---
layout: post
category: Linux
title: Redis Sentinel#4
tagline: by Pigbrain
tags: [Tech]
---
Redis Sentinel#4! <br>
sentinel은 redis 인스턴스들을 어떻게 관리할까?

<!--more-->

\#1과 \#2에서 sentinel 기능은 아래 코드에서 부터 시작한다고 하였다. 
{% highlight c %}
// redis.c
run_with_period(100) {  
	if (server.sentinel_mode)  
		sentinelTimer();
}
{% endhighlight %}	

\#4부터는 sentinelTimer()안에 있는 실질적인 sentinel 코드들을 살펴보도록 하겠다. <br>

{% highlight c %}
// sentinel.c
void sentinelTimer(void) {
 	...
	sentinelHandleDictOfRedisInstances(sentinel.masters);
 	...
}
{% endhighlight %}
이번절에서 살펴볼 함수는 sentinelHandleDictOfRedisInstances 이다. <br>
sentinelHandleDictOfRedisInstances 에서는 sentinel의 거의 모든 기능이 있다고 봐도 무방하다. <br>
예를 들어 주기적으로 인스턴스들(sentinel, redis-master, redis-slave)에게 ping을 보내거나 새로운 sentinel을 발견하기 위해 hello를 보내거나 
혹은 연결이 끊어진 인스턴스들에 대해서는 reconnect를 시도하고 master의 상태(SDOWN/ODWN)에 따라 fail-over를 처리하기도 하고 등등 수 많은 일을 한다.  
	

sentinelHandleDictOfRedisInstances 함수의 구현부 이다. 매 틱마다 인스턴스들에 대하여 동일한 처리한다.
추가로 인스턴스가 redis-master일 경우에만 추가적인 처리를 한다.
{% highlight c %}
// sentinel.c
void sentinelHandleDictOfRedisInstances(dict *instances) {
	dictIterator *di;
	dictEntry *de;
	sentinelRedisInstance *switch_to_promoted = NULL;
	
	di = dictGetIterator(instances);
	while((de = dictNext(di)) != NULL) {
		sentinelRedisInstance *ri = dictGetVal(de);

		sentinelHandleRedisInstance(ri);	
		if (ri->flags & SRI_MASTER) {
			sentinelHandleDictOfRedisInstances(ri->slaves);
			sentinelHandleDictOfRedisInstances(ri->sentinels);
			if (ri->failover_state == SENTINEL_FAILOVER_STATE_UPDATE_CONFIG) {
				switch_to_promoted = ri;
			}
		} 
	}
	if (switch_to_promoted)
		sentinelFailoverSwitchToPromotedSlave(switch_to_promoted);
	dictReleaseIterator(di);
}
{% endhighlight %}


sentinelHandleRedisInstance(..)의 구현부 이다.
{% highlight c %}
// sentinel.c
void sentinelHandleRedisInstance(sentinelRedisInstance *ri) {
	sentinelReconnectInstance(ri);
	sentinelSendPeriodicCommands(ri);

	if (sentinel.tilt) {
		if (mstime()-sentinel.tilt_start_time < SENTINEL_TILT_PERIOD) return;
		sentinel.tilt = 0;
		sentinelEvent(REDIS_WARNING,"-tilt",NULL,"#tilt mode exited");
	}

	sentinelCheckSubjectivelyDown(ri);

	if (ri->flags & (SRI_MASTER|SRI_SLAVE)) {
	}

	if (ri->flags & SRI_MASTER) {
		sentinelCheckObjectivelyDown(ri);
		if (sentinelStartFailoverIfNeeded(ri))
			sentinelAskMasterStateToOtherSentinels(ri,SENTINEL_ASK_FORCED);
		sentinelFailoverStateMachine(ri);
		sentinelAskMasterStateToOtherSentinels(ri,SENTINEL_NO_FLAGS);
	}
}
{% endhighlight %}
sentinelReconnectInstance()에서는 disconnect되어 있는 인스턴스들에 대해서 connect를 시도한다. <br>
connect는 비동기로 처리하며 connect/disconnect에 대한 콜백 함수를 설정한다. <br>
만약 connect 하는 대상이 redis-master/slave일 경우 "\_\_sentinel\_\_:hello" 메세지를 보낸다. <br>
"\_\_sentinel\_\_:hello"메세지를 통하여 sentinel이 다른 sentinel들을 자동으로 발견할수 있다.
아래는 SENTINEL_HELLO_CHANNEL("\_\_sentinel\_\_:hello")을 전송하고 응답 데이터 수신시 sentinelReceiveHelloMessages 
함수를 콜백으로 설정하는 코드이다.
{% highlight c %}
// sentinel.c
void sentinelReconnectInstance(sentinelRedisInstance *ri) {
	...
	if ((ri->flags & (SRI_MASTER|SRI_SLAVE)) && ri->pc == NULL) {
		...
		redisAsyncCommand(ri->pc,
				sentinelReceiveHelloMessages, NULL, "SUBSCRIBE %s",
				SENTINEL_HELLO_CHANNEL); 
		
		...
	}
	...
}
{% endhighlight %}
sentinelReceiveHelloMessages(..) 콜백 함수에는  "\_\_sentinel\_\_:hello"메세지 응답을 실질적으로 처리하는
sentinelProcessHelloMessage(..) 함수가 있다.
{% highlight c %}
void sentinelProcessHelloMessage(char *hello, int hello_len) {
	...
	master = sentinelGetMasterByName(token[4]);
	if (!master) goto cleanup;
	
	port = atoi(token[1]);
	master_port = atoi(token[6]);
	si = getSentinelRedisInstanceByAddrAndRunID(master->sentinels,token[0],port,token[2]);
	current_epoch = strtoull(token[3],NULL,10);
	master_config_epoch = strtoull(token[7],NULL,10);
	
	if (!si) {
		...
		removed = removeMatchingSentinelsFromMaster(master,token[0],port,token[2]);
		if (removed) {
			sentinelEvent(REDIS_NOTICE,"-dup-sentinel",master,  "%@ #duplicate of %s:%d or %s", token[0],port,token[2]);
		}

		si = createSentinelRedisInstance(NULL,SRI_SENTINEL, token[0],port,master->quorum,master);
	}
	...
}
{% endhighlight %}
"\_\_sentinel\_\_:hello"메세지 응답으로 master name, sentinel의 ip, port 등등이있는데 우선
master name으로 현재 자신의 redis-master를 조회한 후 자신이 감시하지 않는 master일 경우 처리하지 않는다. 
(다른 sentinel들을 발견하더라도 자신과 모니터링하는 대상이 동일한 sentinel에 대해서만 연결을 하려는 것으로 보인다.)
그리고 sentinel 리스트에서 ip, port정보로 현재 등록되어 있는 sentinel 인지 검사한 후에 등록되지 않은 sentinel이라면 새로운 sentinel 인스턴스를 생성한다.
만약 새로운 sentinel을 추가하기전에 runid로 중복된 sentinel이 있는지 검사하고 만약 존재한다면 해당 인스턴스를 삭제 한 후 새로운 인스턴스를 생성한다.  


다시 sentinelHandleRedisInstance(..)로 돌아가서 sentinelSendPeriodicCommands(..)를 보자.  
여기서는 매초마다 인스턴스들에게 ping을 보내거나 앞서 얘기한 "\_\_sentinel\_\_:hello"메세지를 보낸다.
("\_\_sentinel\_\_:hello"메세지는 연결되었을때 외에도 주기적으로(약 5분간격)으로 보낸다.)
{% highlight c %}
// sentinel.c
void sentinelSendPeriodicCommands(sentinelRedisInstance *ri) {
 	... 
 	
	if ((ri->flags & SRI_SENTINEL) == 0 &&
		(ri->info_refresh == 0 ||
		(now - ri->info_refresh) > info_period)) {
			retval = redisAsyncCommand(ri->cc,
			sentinelInfoReplyCallback, NULL, "INFO");
			if (retval == REDIS_OK) ri->pending_commands++;
	} else if ((now - ri->last_pong_time) > ping_period) {
		sentinelSendPing(ri);
	} else if ((now - ri->last_pub_time) > SENTINEL_PUBLISH_PERIOD) {
		sentinelSendHello(ri);
	}
	...
}
{% endhighlight %}

sentinelHandleRedisInstance(..) 중 sentinelCheckSubjectivelyDown(..)에서는 ping을 보내고 난 후 지정된 시간안에 응답이 없을 경우 
sentinel은 SDOWN상태로 업데이트 한다. 단 redis-master에 대해서만 SDOWN 상태로 업데이트하고 이 후 sentinel들 끼리 투표를 통하여 ODOWN 상태 업데이트를 결정하며 
sentinel이나 redis-slave에 대해서는 SDOWN 상태가 되면 바로 disconnect를 진행한다.


\<수정중\>


#To be continue....#