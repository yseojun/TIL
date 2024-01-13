>커널에 할당된 폴링 공간에 모니터링할 이벤트를 등록하고, 발생한 이벤트를 return받아 Multiple I/O Event를 처리할 수 있도록 도와준다.
---
- kqueue()
```cpp
#include <sys/time.h>
#include <sys/event.h>
#include <sys/types.h>

int kqueue(void);

return fd;
```
커널에 새로운 `event queue`를 만들고, fd를 return
return된 fd는 kevent()에서 이벤트를 등록, 모니터링

- kevent()
```cpp
int kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);

return event_count;
```
새로 모니터링할 이벤트를 등록하고, 아직 처리되지 않은 event의 개수를 return 한다.


```cpp
...

newEventsCount = kevent(this->kqueueFd, this->subscribedEvents.data(), this->subscribedEvents.size(), newEvents, SIZE_EVENT_POOL, NULL);

// newEvents에 퍼내고 남은 이벤트를 정리.
this->subscribedEvents.clear();
if (newEventsCount == SYS_FAIL) {
	throw RUNTIME_ERROR("Error : Failed to run kevent system call!");
	this->isRunning = false;
	break;
}

for (int i = 0; i < newEventsCount; i++) {
	struct kevent cur = newEvents[i];
	if (cur.flags & EV_ERROR) {
		if (isServerEvent(cur)) { // static_cast<int>(event.ident) == this->socketFd
			throw RUNTIME_ERROR("Error : Internal Server Error!");
			this->isRunning = false;
			break;
	} else {
		this->rejectClient(cur);
	}
	} else if (cur.filter & EVFILT_READ) {
		if (isServerEvent(cur)) { // static_cast<int>(event.ident) == this->socketFd
			this->acceptClient(cur);
		} else {
		if (this->containsEvent(cur) == false) { // this->socketClientMap.find(event.ident) == this->socketClientMap.end()
		continue;
	}
		this->socketClientMap[cur.ident]->markAsModifiedAt(DateUtil::now());
		this->handleReadEvent(cur);
	}
	} else if (cur.filter & EVFILT_WRITE) {
		if (this->containsEvent(cur) == false) { // this->socketClientMap.find(event.ident) == this->socketClientMap.end()
		continue;
	}
	this->socketClientMap[cur.ident]->markAsModifiedAt(DateUtil::now());
	this->handleWriteEvent(cur);
}
...
```

kevent의 `filter` 에서 이벤트 형식을 파악하고 형식에 맞게 수행