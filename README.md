Android Basic Fragment Example
==============================

Simple example of a 2d application on android using canvas

[![ScreenShot](https://raw.github.com/joninvski/android_graphics_example/master/images/bubbleYoutubeScreenshot.png)](http://youtu.be/nNqc57o9UCo)


Compile
-------

    # Optional
    ANDROID_HOME=/home/.../android/sdk; export ANDROID_HOME

    ./gradlew compileDebug

Test
----

    # Make sure emulator is running or connected to real device
    ./gradlew connectedInstrumentTest


Code highlights
---------------

### Sound

    onResume() {
		mAudioManager = (AudioManager) getSystemService(AUDIO_SERVICE);
		mStreamVolume = (float) mAudioManager.getStreamVolume(AudioManager.STREAM_MUSIC)
				/ mAudioManager.getStreamMaxVolume(AudioManager.STREAM_MUSIC);

		// TODO - make a new SoundPool, allowing up to 10 streams
		int MAX_STREAMS = 10;
		mSoundPool = new SoundPool(MAX_STREAMS, AudioManager.STREAM_MUSIC, 0);

		// TODO - set a SoundPool OnLoadCompletedListener that calls setupGestureDetector()
		mSoundPool.setOnLoadCompleteListener(new OnLoadCompleteListener() {
			@Override
			public void onLoadComplete(SoundPool soundPool, int sampleId, int status) {
                // Start something
			}
		});

		// TODO - load the sound from res/raw/bubble_pop.wav
		mSoundID = mSoundPool.load(this, R.raw.bubble_pop, 1);

		mAudioManager.setSpeakerphoneOn(true);
		mAudioManager.loadSoundEffects();
    }


    onPause() {
		// Release all SoundPool resources
		if (null != mSoundPool) {
			mSoundPool.unload(mSoundID);
			mSoundPool.release();
			mSoundPool = null;
		}
		mAudioManager.setSpeakerphoneOn(false);
		mAudioManager.unloadSoundEffects();
    }

    // On bubble pop
    ...
    mSoundPool.play(mSoundID, mStreamVolume, mStreamVolume, 1, 0, 1f);
    ....

### Gesture detection

	private void setupGestureDetector() {
		mGestureDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener() {

                @Override
                public boolean onFling(MotionEvent event1, MotionEvent event2, float velocityX, float velocityY) {
                    ...
                    return true; // Event was treated
                }

                @Override
                public boolean onSingleTapConfirmed(MotionEvent event) {
                    int pointerIndex = event.getActionIndex();
                    int pointerID = event.getPointerId(pointerIndex);

                    float x = event.getX(pointerID);
                    float y = event.getY(pointerID);
                    ...
                    return true;
                }
		});
	}

	@Override
	public boolean onTouchEvent(MotionEvent event) {
		// TODO - delegate the touch to the gestureDetector
		return mGestureDetector.onTouchEvent(event);
	}

### Graphics

	private void addBubble(float x, float y) {
		BubbleView child = new BubbleView(getApplicationContext(), x, y);
		mFrame.addView(child);
		child.start();
	}

    private void createScaledBitmap(Random r) {
        mScaledBitmapWidth = (r.nextInt(3) + 1) * BITMAP_SIZE;
        mScaledBitmap = Bitmap.createScaledBitmap(mBitmap, mScaledBitmapWidth, mScaledBitmapWidth, false);
    }

    @Override
    protected synchronized void onDraw(Canvas canvas) {
        // TODO - save the canvas
        canvas.save();

        // TODO - increase the rotation of the original image by mDRotate
        mRotate = mRotate + mDRotate;

        // TODO Rotate the canvas by current rotation
        canvas.rotate(mRotate, mXPos + mScaledBitmapWidth / 2, mYPos + mScaledBitmapWidth / 2);

        // TODO - draw the bitmap at it's new location
        canvas.drawBitmap(mScaledBitmap, this.mXPos, this.mYPos, null);

        // TODO - restore the canvas
        canvas.restore();
    }

    // Start moving the BubbleView & updating the display
    private void start() {
        // Creates a WorkerThread
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

        // Execute the run() in Worker Thread every REFRESH_RATE milliseconds
        // Save reference to this job in mMoverFuture
        mMoverFuture = executor.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                boolean onScreen = !moveWhileOnScreen();
                if (onScreen) {
                    // postinvalidate because non-ui thread
                    postInvalidate(); 
                } else { // Bubble out of bounds
                    stop(false);
                }
            }
        }, 0, REFRESH_RATE, TimeUnit.MILLISECONDS);
    }

    private void stop(final boolean popped) {
        if (null != mMoverFuture && mMoverFuture.cancel(true)) {
            // This work will be performed on the UI Thread
            mFrame.post(new Runnable() {
                @Override
                public void run() {
                    // TODO - Remove the BubbleView from mFrame
                    mFrame.removeView(BubbleView.this);
                }
            });
        }
    }
