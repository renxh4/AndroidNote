
# Fps采集

# 卡顿时间

# 卡顿堆栈采集



# Choreographer

先上代码
```java
## ViewRootImpl.java

  void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            //发送同步屏障
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            //
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
```
`mChoreographer`翻译为编舞者,用于接受系统的`VSync`信号,在下一个帧渲染时控制一些操作，`mChoreographer`的`postCallback`方法用于添加回调，这个添加的回调，将在下一帧渲染时执行，这个添加的回调指的是`TraversalRunnabl`e类型的`mTraversalRunnable`，如下：


```java
final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }

```

这个方法内部调用了doTraversal
```java
   void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```
这个方法又调用了`performTraversals`,这个方法中更新了`Window`的视图，并且完成了`View`的绘制流程，`measure，layout，draw`,这样就完成了`View`的更新

## 继续追一下mChoreographer.postCallback

```java
## Choreographer.java
public void postCallback(int callbackType, Runnable action, Object token) {
        postCallbackDelayed(callbackType, action, token, 0);
    }


 public void postCallbackDelayed(int callbackType,
            Runnable action, Object token, long delayMillis) {
        if (action == null) {
            throw new IllegalArgumentException("action must not be null");
        }
        if (callbackType < 0 || callbackType > CALLBACK_LAST) {
            throw new IllegalArgumentException("callbackType is invalid");
        }

        postCallbackDelayedInternal(callbackType, action, token, delayMillis);
    }


private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
        if (DEBUG_FRAMES) {
            Log.d(TAG, "PostCallback: type=" + callbackType
                    + ", action=" + action + ", token=" + token
                    + ", delayMillis=" + delayMillis);
        }

        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;
            //把Runnable加入数组中
            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

            if (dueTime <= now) {
                //立即执行
                scheduleFrameLocked(now);
            } else {
                //发送异步消息延迟执行
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }
```

```java
 private void scheduleFrameLocked(long now) {
        if (!mFrameScheduled) {
            mFrameScheduled = true;
            if (USE_VSYNC) {//// Android 4.1 之后 USE_VSYNCUSE_VSYNC 默认为 true
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "Scheduling next frame on vsync.");
                }

                // 如果再主线程，则立即执行，否则就发送异步消息
                if (isRunningOnLooperThreadLocked()) {
                    scheduleVsyncLocked();
                } else {
                    Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                    msg.setAsynchronous(true);
                    mHandler.sendMessageAtFrontOfQueue(msg);
                }
            } else {// 未开启 vsync，4.1 之后默认开启
                final long nextFrameTime = Math.max(
                        mLastFrameTimeNanos / TimeUtils.NANOS_PER_MS + sFrameDelay, now);
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "Scheduling next frame in " + (nextFrameTime - now) + " ms.");
                }
                Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, nextFrameTime);
            }
        }
    }
```
这里就是判断是否开启`USE_VSYNC`,如果是主线程则立即执行
```java
 private void scheduleVsyncLocked() {
        mDisplayEventReceiver.scheduleVsync();
    }
```
`VSYNC`其实是为了解决，屏幕刷新率和GPU帧率不一致导致的屏幕撕裂问题，`VSYNC`机制再Android4.1引入,简单理解为，VSYNC是硬件发送的定时信号，通过Choreographer来监听这个信号，每当信号来临，开始绘制工作

`mDisplayEventReceiver` 类型是`FrameDisplayEventReceiver`其父类为`DisplayEventReceiver`,`scheduleVsync`方法为父类的方法
```java
## DisplayEventReceiver.java

 public void scheduleVsync() {
        if (mReceiverPtr == 0) {
            Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                    + "receiver has already been disposed.");
        } else {
            //注册Vsync信号
            nativeScheduleVsync(mReceiverPtr);
        }
    }

//有 vsync 信号时，由 native 调用此方法
private void dispatchVsync(long timestampNanos, long physicalDisplayId, int frame) {
        onVsync(timestampNanos, physicalDisplayId, frame);
    }
```

这里是调用了`native`方法`nativeScheduleVsync`注册信号，当有Vsync信号时回调`dispatchVsync`方法，然后调用`onVsync`方法，这个方法是由子类`FrameDisplayEventReceiver`实现

```java

    private final class FrameDisplayEventReceiver extends DisplayEventReceiver
            implements Runnable {
        private boolean mHavePendingVsync;
        private long mTimestampNanos;
        private int mFrame;

        public FrameDisplayEventReceiver(Looper looper, int vsyncSource) {
            super(looper, vsyncSource);
        }

       
        @Override
        public void onVsync(long timestampNanos, long physicalDisplayId, int frame) {
           
            long now = System.nanoTime();
            if (timestampNanos > now) {
                Log.w(TAG, "Frame time is " + ((timestampNanos - now) * 0.000001f)
                        + " ms in the future!  Check that graphics HAL is generating vsync "
                        + "timestamps using the correct timebase.");
                timestampNanos = now;
            }

            if (mHavePendingVsync) {
                Log.w(TAG, "Already have a pending vsync event.  There should only be "
                        + "one at a time.");
            } else {
                mHavePendingVsync = true;
            }

            mTimestampNanos = timestampNanos;
            mFrame = frame;
            //这里传入this，回掉自身的run方法
            Message msg = Message.obtain(mHandler, this);
            //发送异步消息
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
        }

        @Override
        public void run() {
            mHavePendingVsync = false;
            doFrame(mTimestampNanos, mFrame);
        }
    }
```
这里`sendMessageAtTime`方法的参数`timestampNanos / TimeUtils.NANOS_PER_MS`,`timestampNanos`表示vysnc信号时间戳，单位纳秒，这里转换成毫秒，此时Vysnc已经发生，`timestampNanos`是比当前时间小的，这样消息就可以塞到MessageQueue的前面，此处的this，表示回调自身的`run`方法,最后调用`doFrame`

```java
    void doFrame(long frameTimeNanos, int frame) {
        final long startNanos;
        synchronized (mLock) {
            if (!mFrameScheduled) {
                return; // no work to do
            }
            //
            long intendedFrameTimeNanos = frameTimeNanos;
            startNanos = System.nanoTime();
            final long jitterNanos = startNanos - frameTimeNanos;
            //startNanos当前时间，frameTimeNanos表示Vysnc信号时间，相减得到主线程耗时时间
            if (jitterNanos >= mFrameIntervalNanos) {
                // mFrameIntervalNanos表示一帧时间
                final long skippedFrames = jitterNanos / mFrameIntervalNanos;
                if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {
                    //掉帧抄超过30帧打印log
                    Log.i(TAG, "Skipped " + skippedFrames + " frames!  "
                            + "The application may be doing too much work on its main thread.");
                }
                final long lastFrameOffset = jitterNanos % mFrameIntervalNanos;
                if (DEBUG_JANK) {
                    Log.d(TAG, "Missed vsync by " + (jitterNanos * 0.000001f) + " ms "
                            + "which is more than the frame interval of "
                            + (mFrameIntervalNanos * 0.000001f) + " ms!  "
                            + "Skipping " + skippedFrames + " frames and setting frame "
                            + "time to " + (lastFrameOffset * 0.000001f) + " ms in the past.");
                }
                frameTimeNanos = startNanos - lastFrameOffset;
            }

            if (frameTimeNanos < mLastFrameTimeNanos) {
                if (DEBUG_JANK) {
                    Log.d(TAG, "Frame time appears to be going backwards.  May be due to a "
                            + "previously skipped frame.  Waiting for next vsync.");
                }
                scheduleVsyncLocked();
                return;
            }

            if (mFPSDivisor > 1) {
                long timeSinceVsync = frameTimeNanos - mLastFrameTimeNanos;
                if (timeSinceVsync < (mFrameIntervalNanos * mFPSDivisor) && timeSinceVsync > 0) {
                    scheduleVsyncLocked();
                    return;
                }
            }

            mFrameInfo.setVsync(intendedFrameTimeNanos, frameTimeNanos);
            mFrameScheduled = false;
            mLastFrameTimeNanos = frameTimeNanos;
        }

        try {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "Choreographer#doFrame");
            AnimationUtils.lockAnimationClock(frameTimeNanos / TimeUtils.NANOS_PER_MS);

            //开始回调callback
            mFrameInfo.markInputHandlingStart();
            doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);

            mFrameInfo.markAnimationsStart();
            doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
            doCallbacks(Choreographer.CALLBACK_INSETS_ANIMATION, frameTimeNanos);

            mFrameInfo.markPerformTraversalsStart();
            doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);

            doCallbacks(Choreographer.CALLBACK_COMMIT, frameTimeNanos);
        } finally {
            AnimationUtils.unlockAnimationClock();
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }


    }
```
下面调用`doCallbacks`方法

```java
  void doCallbacks(int callbackType, long frameTimeNanos) {
        CallbackRecord callbacks;
        synchronized (mLock) {
         
            final long now = System.nanoTime();
            //根据callbackType找到callbacks对象
            callbacks = mCallbackQueues[callbackType].extractDueCallbacksLocked(
                    now / TimeUtils.NANOS_PER_MS);
            if (callbacks == null) {
                return;
            }
            mCallbacksRunning = true;

        ...
          
        try {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, CALLBACK_TRACE_TITLES[callbackType]);
            for (CallbackRecord c = callbacks; c != null; c = c.next) {
                if (DEBUG_FRAMES) {
                    Log.d(TAG, "RunCallback: type=" + callbackType
                            + ", action=" + c.action + ", token=" + c.token
                            + ", latencyMillis=" + (SystemClock.uptimeMillis() - c.dueTime));
                }
                //执行callback的run方法
                c.run(frameTimeNanos);
            }
        } 
        ...
    }
```
根据`callbackType`找到`callbacks`对象,然后执行`run`方法,`callbackType`有四种 `CALLBACK_INPUT`、`CALLBACK_ANIMATION`、`CALLBACK_TRAVERSAL`、`CALLBACK_COMMIT `


```java
private static final class CallbackRecord {
   
        public void run(long frameTimeNanos) {
            if (token == FRAME_CALLBACK_TOKEN) {
                //注意这里我们自定义的就是FrameCallback，回调这里
                ((FrameCallback)action).doFrame(frameTimeNanos);
            } else {
                //到这里就执行了Runnable的run方法
                ((Runnable)action).run();
            }
        }
    }
```

到这里就执行了`Runnable`的`run`方法

## 如何监测应用的 FPS？

其实这里就应用到了一个

```java
> Choreographer.java

public void postFrameCallback(FrameCallback callback) {
    postFrameCallbackDelayed(callback, 0);
}

public void postFrameCallbackDelayed(FrameCallback callback, long delayMillis) {
    ......
    // 这里的类型是 CALLBACK_ANIMATION
    postCallbackDelayedInternal(CALLBACK_ANIMATION,
            callback, FRAME_CALLBACK_TOKEN, delayMillis);
}
```

public void postFrameCallback(FrameCallback callback) {
public void postFrameCallback(FrameCallback callback) {
这里可以看到这个方法和上面的`postCallback`一样，差别就是`callbackType`,这个的`callType`是`CALLBACK_ANIMATION`，也就是说每次Vysnc回调都会调用我们自定义的`FrameCallback`的`doFrame`方法，我们可以再这个方法中计算fps

## 实战

```java
public class Fps1 {

    private boolean isFpsOpen;
    private ArrayList<Integer> listeners;
    private int count;
    private Handler handler;
    private long FPS_INTERVAL_TIME = 1000L;
    private FpsCallback mfpsCallback;
    private MyCallback fpsRunnable;


    public Fps1() {
        handler = new Handler(Looper.getMainLooper());
    }


    public void startMonitor(FpsCallback fpsCallback) {
        // 防止重复开启
        if (!isFpsOpen) {
            isFpsOpen = true;
            this.mfpsCallback = fpsCallback;
            fpsRunnable = new MyCallback();
            handler.postDelayed(fpsRunnable, FPS_INTERVAL_TIME);
            Choreographer.getInstance().postFrameCallback(fpsRunnable);
        }
    }

    public void stopMonitor() {
        count = 0;
        handler.removeCallbacks(fpsRunnable);
        Choreographer.getInstance().removeFrameCallback(fpsRunnable);
        isFpsOpen = false;
    }


    class MyCallback implements Choreographer.FrameCallback, Runnable {

        @Override
        public void doFrame(long frameTimeNanos) {
            count++;
            Choreographer.getInstance().postFrameCallback(this);
        }

        @Override
        public void run() {
            mfpsCallback.fps(count);
            count = 0;
            handler.postDelayed(this, FPS_INTERVAL_TIME);
        }
    }

    interface FpsCallback {
        void fps(int fps);
    }
}
```







