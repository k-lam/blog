##In Activity

    final FragmentManagerImpl mFragments = new FragmentManagerImpl();
    final FragmentContainer mContainer = new FragmentContainer() {
        @Override
        public View findViewById(int id) {
            return Activity.this.findViewById(id);
        }
    };

其中 

	/**
 	* Callbacks from FragmentManagerImpl to its container.
 	*/
	interface FragmentContainer {
    	public View findViewById(int id);
	}

FragmentManager见下文：

##FragmentManager
Interface for interacting with Fragment objects inside of an Activity
也就是说，FragementManager只是一个桥梁,连接Activity和Fragment

##FragmentTransaction
API for performing a set of Fragment operations.

这是一个抽象类，实现类是

	final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, Runnable 

其中FragmentManager.BackStackEntry的说明是

	 /**
     * *Representation of an entry on the fragment back stack*, as created
     * with {@link FragmentTransaction#addToBackStack(String)
     * FragmentTransaction.addToBackStack()}.  Entries can later be
     * retrieved with {@link FragmentManager#getBackStackEntryAt(int)
     * FragmentManager.getBackStackEntry()}.
     *
     * <p>Note that you should never hold on to a BackStackEntry object;
     * the identifier as returned by {@link #getId} is the only thing that
     * will be persisted across activity instances.
     */ 

可以看出，这个BackStackRecord除了是a set of Fragment operations，还实现了记录这些sets of Fragment operations

从这个BackStackRecord开始分析。

##从官网的一个例子开始
这个例子是 replace 一个fragment

	FragmentTransaction ft = getFragmentManager().beginTransaction();
    ft.replace(R.id.simple_fragment, newFragment);
    ft.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
    ft.addToBackStack(null);
    ft.commit();




1.	**FragmentTransaction ft = getFragmentManager().beginTransaction();**

	    public FragmentTransaction beginTransaction() {
	        return new BackStackRecord(this);
	    } 


	实际上就是new一个BackStackRecord，且把FragmentManager的引用mManager给BackStackRecord。

2.	**ft.replace(R.id.simple_fragment, newFragment);**

	    public FragmentTransaction replace(int containerViewId, Fragment fragment) {
	        return replace(containerViewId, fragment, null);
	    }

	    public FragmentTransaction replace(int containerViewId, Fragment fragment, String tag) {
	        if (containerViewId == 0) {
	            throw new IllegalArgumentException("Must use non-zero containerViewId");
	        }
	
	        doAddOp(containerViewId, fragment, tag, OP_REPLACE);
	        return this;
	    }

	留意到doAddOp这个方法，其中最后一个参数OP_REPLACE
	
		static final int OP_NULL = 0;
	    static final int OP_ADD = 1;
	    static final int OP_REPLACE = 2;
	    static final int OP_REMOVE = 3;
	    static final int OP_HIDE = 4;
	    static final int OP_SHOW = 5;
	    static final int OP_DETACH = 6;
	    static final int OP_ATTACH = 7;
	
	其实就类似于枚举了

	*doAddOp*

	    private void doAddOp(int containerViewId, Fragment fragment, String tag, int opcmd) {
	        fragment.mFragmentManager = mManager;
			......
	        Op op = new Op();
	        op.cmd = opcmd;
	        op.fragment = fragment;
	        addOp(op);
		}

	fragment.mFragmentManager = mManager; 这样 fragment就保存了FragmentManager的引用了。
	FragmentManager作为Activity和Fragment的桥梁，现在桥梁的两端都保存了FragmentManager的引用
	
	*Op*
	
	    static final class Op {
	        Op next;
	        Op prev;
	        int cmd;
	        Fragment fragment;//当前Fragment
	        int enterAnim;
	        int exitAnim;
	        int popEnterAnim;
	        int popExitAnim;
	        ArrayList<Fragment> removed;
	    }
	
	就是Operation
	
		**addOp**
	
	    void addOp(Op op) {
	        if (mHead == null) {
	            mHead = mTail = op;
	        } else {
	            op.prev = mTail;
	            mTail.next = op;
	            mTail = op;
	        }
	        op.enterAnim = mEnterAnim;
	        op.exitAnim = mExitAnim;
	        op.popEnterAnim = mPopEnterAnim;
	        op.popExitAnim = mPopExitAnim;
	        mNumOp++;
	    }
	
	这个...不就是一个链表吗？就是一个链表mHead和mTail都是Op。也就是说，一连串的操作组成的事务（Transcation），是用这个链表来存储的，有头指针和方便操作的尾指针。
	
	而且可以知道，其他操作，delete，add，show,hide这些都会是把op加到这个链表中。

3.	**ft.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);**没啥好看，就是设置一下动画。

4.	**ft.addToBackStack(null);**

	    public FragmentTransaction addToBackStack(String name) {
	        if (!mAllowAddToBackStack) {
	            throw new IllegalStateException(
	                    "This FragmentTransaction is not allowed to be added to the back stack.");
	        }
	        mAddToBackStack = true;
	        mName = name;
	        return this;
	    }

留意到，mAddToBackStack是整个BackStackRecord类的，不属于某个Op，也就是说，Back Stack中的entity指的是Transcation，而不是单一个Operation

5.	**ft.commit();**
	
	    public int commit() {
	        return commitInternal(false);
	    }
	
	    int commitInternal(boolean allowStateLoss) {
	        if (mCommitted) throw new IllegalStateException("commit already called");
	        if (FragmentManagerImpl.DEBUG) {
	            Log.v(TAG, "Commit: " + this);
	            LogWriter logw = new LogWriter(Log.VERBOSE, TAG);
	            PrintWriter pw = new PrintWriter(logw);
	            dump("  ", null, pw, null);
	        }
	        mCommitted = true;
	        if (mAddToBackStack) {
	            mIndex = mManager.allocBackStackIndex(this);
	        } else {
	            mIndex = -1;
	        }
	        mManager.enqueueAction(this, allowStateLoss);
	        return mIndex;
	    }

	我们先看看commit()返回值的官网解析：`Returns the identifier of this transaction's back stack entry, if addToBackStack(String) had been called. Otherwise, returns a negative number`
	这个返回值如何用呢？
	直接给出答案吧，`FragmentManager.popBackStack (int id, int flags)`

	既然这样，那必须要在FragmentManager中保存一个键值对，identifier->BackStackRecord对象。作为以后能popBackStack用。mIndex = mManager.allocBackStackIndex(this)就是保存并返回identifier的
	
		
		FragmentManagerImpl.allocBackStackIndex:
	    public int allocBackStackIndex(BackStackRecord bse) {
	        synchronized (this) {
	            if (mAvailBackStackIndices == null || mAvailBackStackIndices.size() <= 0) {
	                if (mBackStackIndices == null) {
	                    mBackStackIndices = new ArrayList<BackStackRecord>();
	                }
	                int index = mBackStackIndices.size();
	                if (DEBUG) Log.v(TAG, "Setting back stack index " + index + " to " + bse);
	                mBackStackIndices.add(bse);
	                return index;
	
	            } else {
	                int index = mAvailBackStackIndices.remove(mAvailBackStackIndices.size()-1);
	                if (DEBUG) Log.v(TAG, "Adding back stack index " + index + " with " + bse);
	                mBackStackIndices.set(index, bse);
	                return index;
	            }
	        }
	    }

	这里比较复杂，怎么办？我们从简单的分析起，假设
	
		//ftaddBackStack(null)；//不执行这句，直接commit
		ft.commit();


	*FragmentManagerImpl.enqueueAction*
		
		public void enqueueAction(Runnable action, boolean allowStateLoss) {
	        ...
	            mPendingActions.add(action);
	            if (mPendingActions.size() == 1) {
	                mActivity.mHandler.removeCallbacks(mExecCommit);
	                mActivity.mHandler.post(mExecCommit);
	            }
	    }

		Runnable mExecCommit = new Runnable() {
	        @Override
	        public void run() {
	            execPendingActions();
	        }
    	};

		
	
	mPendingActions是一个`ArrayList<Runnable>`当size == 1时，立刻执行。不是的话，就交给Activity的Handler去schedule。回忆Handler的机制，就能知道，**不是立刻执行的！！！**

	可以看到，真正的执行方法在`execPendingActions()`
	
	    /**
	     * Only call from main thread!
	     */
	    public boolean execPendingActions() {

	        ......

	        boolean didSomething = false;
	
	        while (true) {
	            int numActions;
	            //这个同步控制块是为了把mPendingActions的Runnable都放到mTmpActions
				//这个数组中，释放mPendingActions这个出来继续接收新的Runnable
				//也就是，这次只run目前为止mPendingActions中的Runnable，直到
				//mPendingActions为空
	            synchronized (this) {
	                if (mPendingActions == null || mPendingActions.size() == 0) {
	                    break;
	                }
	                
	                numActions = mPendingActions.size();
	                if (mTmpActions == null || mTmpActions.length < numActions) {
	                    mTmpActions = new Runnable[numActions];
	                }
	                mPendingActions.toArray(mTmpActions);
	                mPendingActions.clear();
	                mActivity.mHandler.removeCallbacks(mExecCommit);
	            }
	            
				//执行
	            mExecutingActions = true;
	            for (int i=0; i<numActions; i++) {
	                mTmpActions[i].run();
	                mTmpActions[i] = null;
	            }
	            mExecutingActions = false;
	            didSomething = true;
	        }
			
			//有延迟开始的,执行以下代码
	        if (mHavePendingDeferredStart) {
	            boolean loadersRunning = false;
	            for (int i=0; i<mActive.size(); i++) {
	                Fragment f = mActive.get(i);
	                if (f != null && f.mLoaderManager != null) {
	                    loadersRunning |= f.mLoaderManager.hasRunningLoaders();
	                }
	            }
	            if (!loadersRunning) {
	                mHavePendingDeferredStart = false;
	                startPendingDeferredFragments();
	            }
	        }
	        return didSomething;
	    }

	对于最后一块代码，奇怪了，延迟开始又是什么？run都run完了，由于研究这块代码有困难，我们暂且放一放，先记住。

	现在回来看BackStackRecord的run方法

		public void run() {

	        ...

	        Op op = mHead;
	        while (op != null) {
	            switch (op.cmd) {
	                case OP_ADD: {
	                    Fragment f = op.fragment;
	                    f.mNextAnim = op.enterAnim;
	                    mManager.addFragment(f, false);
	                } break;
	                case OP_REPLACE: {
	                    Fragment f = op.fragment;
	                    if (mManager.mAdded != null) {
	                        for (int i=0; i<mManager.mAdded.size(); i++) {
	                            Fragment old = mManager.mAdded.get(i);
	                            if (FragmentManagerImpl.DEBUG) Log.v(TAG,
	                                    "OP_REPLACE: adding=" + f + " old=" + old);
	                            if (f == null || old.mContainerId == f.mContainerId) {
	                                if (old == f) {
	                                    op.fragment = f = null;
	                                } else {
	                                    if (op.removed == null) {
	                                        op.removed = new ArrayList<Fragment>();
	                                    }
	                                    op.removed.add(old);
	                                    old.mNextAnim = op.exitAnim;
	                                    if (mAddToBackStack) {
	                                        old.mBackStackNesting += 1;
	                                        if (FragmentManagerImpl.DEBUG) Log.v(TAG, "Bump nesting of "
	                                                + old + " to " + old.mBackStackNesting);
	                                    }
	                                    mManager.removeFragment(old, mTransition, mTransitionStyle);
	                                }
	                            }
	                        }
	                    }
	                    if (f != null) {
	                        f.mNextAnim = op.enterAnim;
	                        mManager.addFragment(f, false);
	                    }
	                } break;
	                case OP_REMOVE: {
	                    Fragment f = op.fragment;
	                    f.mNextAnim = op.exitAnim;
	                    mManager.removeFragment(f, mTransition, mTransitionStyle);
	                } break;
	                case OP_HIDE: {
	                    Fragment f = op.fragment;
	                    f.mNextAnim = op.exitAnim;
	                    mManager.hideFragment(f, mTransition, mTransitionStyle);
	                } break;
	                case OP_SHOW: {
	                    Fragment f = op.fragment;
	                    f.mNextAnim = op.enterAnim;
	                    mManager.showFragment(f, mTransition, mTransitionStyle);
	                } break;
	                case OP_DETACH: {
	                    Fragment f = op.fragment;
	                    f.mNextAnim = op.exitAnim;
	                    mManager.detachFragment(f, mTransition, mTransitionStyle);
	                } break;
	                case OP_ATTACH: {
	                    Fragment f = op.fragment;
	                    f.mNextAnim = op.enterAnim;
	                    mManager.attachFragment(f, mTransition, mTransitionStyle);
	                } break;
	                default: {
	                    throw new IllegalArgumentException("Unknown cmd: " + op.cmd);
	                }
	            }
	
	            op = op.next;
	        }
	
	        mManager.moveToState(mManager.mCurState, mTransition,
	                mTransitionStyle, true);
	
	        if (mAddToBackStack) {
	            mManager.addBackStackState(this);
	        }
	    }

	很简单了，就是执行Transcation中的各个Op，之前说过的Transcation是Op组成的链表。记得我们是选replace来研究的。replace也很简单。在manage中把old的fragment找到，放到Op.removed数组中，在manage中删掉，把当前的addFragment
	
	最后，还有`mManager.moveToState(mManager.mCurState, mTransition, mTransitionStyle, true);`
	

		void moveToState(int newState, int transit, int transitStyle, boolean always) {
	        if (mActivity == null && newState != Fragment.INITIALIZING) {
	            throw new IllegalStateException("No activity");
	        }
	
	        if (!always && mCurState == newState) {
	            return;
	        }
	
	        mCurState = newState;
	        if (mActive != null) {
	            boolean loadersRunning = false;
	            for (int i=0; i<mActive.size(); i++) {
	                Fragment f = mActive.get(i);
	                if (f != null) {
	                    moveToState(f, newState, transit, transitStyle, false);
	                    if (f.mLoaderManager != null) {
	                        loadersRunning |= f.mLoaderManager.hasRunningLoaders();
	                    }
	                }
	            }
	
	            if (!loadersRunning) {
	                startPendingDeferredFragments();
	            }
	
	            if (mNeedMenuInvalidate && mActivity != null && mCurState == Fragment.RESUMED) {
	                mActivity.invalidateOptionsMenu();
	                mNeedMenuInvalidate = false;
	            }
	        }
	    }

	而state又是什么？在Fragment中
	
	    static final int INVALID_STATE = -1;   // Invalid state used as a null value.
	    static final int INITIALIZING = 0;     // Not yet created.
	    static final int CREATED = 1;          // Created.
	    static final int ACTIVITY_CREATED = 2; // The activity has finished its creation.
	    static final int STOPPED = 3;          // Fully created, not started.
	    static final int STARTED = 4;          // Created and started, not resumed.
	    static final int RESUMED = 5;          // Created started and resumed.

	真正的moveToState

	    void moveToState(Fragment f, int newState, int transit, int transitionStyle,
            boolean keepActive) {
	        if (DEBUG && false) Log.v(TAG, "moveToState: " + f
	            + " oldState=" + f.mState + " newState=" + newState
	            + " mRemoving=" + f.mRemoving + " Callers=" + Debug.getCallers(5));
	
	        // Fragments that are not currently added will sit in the onCreate() state.
	        if ((!f.mAdded || f.mDetached) && newState > Fragment.CREATED) {
	            newState = Fragment.CREATED;
	        }
	        if (f.mRemoving && newState > f.mState) {
	            // While removing a fragment, we can't change it to a higher state.
	            newState = f.mState;
	        }
	        // Defer start if requested; don't allow it to move to STARTED or higher
	        // if it's not already started.
	        if (f.mDeferStart && f.mState < Fragment.STARTED && newState > Fragment.STOPPED) {
	            newState = Fragment.STOPPED;
	        }
	        if (f.mState < newState) {
	            // For fragments that are created from a layout, when restoring from
	            // state we don't want to allow them to be created until they are
	            // being reloaded from the layout.
	            if (f.mFromLayout && !f.mInLayout) {
	                return;
	            }
	            if (f.mAnimatingAway != null) {
	                // The fragment is currently being animated...  but!  Now we
	                // want to move our state back up.  Give up on waiting for the
	                // animation, move to whatever the final state should be once
	                // the animation is done, and then we can proceed from there.
	                f.mAnimatingAway = null;
	                moveToState(f, f.mStateAfterAnimating, 0, 0, true);
	            }
	            switch (f.mState) {
	                case Fragment.INITIALIZING:
	                    if (DEBUG) Log.v(TAG, "moveto CREATED: " + f);
	                    if (f.mSavedFragmentState != null) {
	                        f.mSavedViewState = f.mSavedFragmentState.getSparseParcelableArray(
	                                FragmentManagerImpl.VIEW_STATE_TAG);
	                        f.mTarget = getFragment(f.mSavedFragmentState,
	                                FragmentManagerImpl.TARGET_STATE_TAG);
	                        if (f.mTarget != null) {
	                            f.mTargetRequestCode = f.mSavedFragmentState.getInt(
	                                    FragmentManagerImpl.TARGET_REQUEST_CODE_STATE_TAG, 0);
	                        }
	                        f.mUserVisibleHint = f.mSavedFragmentState.getBoolean(
	                                FragmentManagerImpl.USER_VISIBLE_HINT_TAG, true);
	                        if (!f.mUserVisibleHint) {
	                            f.mDeferStart = true;
	                            if (newState > Fragment.STOPPED) {
	                                newState = Fragment.STOPPED;
	                            }
	                        }
	                    }
	                    f.mActivity = mActivity;
	                    f.mParentFragment = mParent;
	                    f.mFragmentManager = mParent != null
	                            ? mParent.mChildFragmentManager : mActivity.mFragments;
	                    f.mCalled = false;
	                    f.onAttach(mActivity);
	                    if (!f.mCalled) {
	                        throw new SuperNotCalledException("Fragment " + f
	                                + " did not call through to super.onAttach()");
	                    }
	                    if (f.mParentFragment == null) {
	                        mActivity.onAttachFragment(f);
	                    }
	
	                    if (!f.mRetaining) {
	                        f.performCreate(f.mSavedFragmentState);
	                    }
	                    f.mRetaining = false;
	                    if (f.mFromLayout) {
	                        // For fragments that are part of the content view
	                        // layout, we need to instantiate the view immediately
	                        // and the inflater will take care of adding it.
	                        f.mView = f.performCreateView(f.getLayoutInflater(
	                                f.mSavedFragmentState), null, f.mSavedFragmentState);
	                        if (f.mView != null) {
	                            f.mView.setSaveFromParentEnabled(false);
	                            if (f.mHidden) f.mView.setVisibility(View.GONE);
	                            f.onViewCreated(f.mView, f.mSavedFragmentState);
	                        }
	                    }
	                case Fragment.CREATED:
	                    if (newState > Fragment.CREATED) {
	                        if (DEBUG) Log.v(TAG, "moveto ACTIVITY_CREATED: " + f);
	                        if (!f.mFromLayout) {
	                            ViewGroup container = null;
	                            if (f.mContainerId != 0) {
	                                container = (ViewGroup)mContainer.findViewById(f.mContainerId);
	                                if (container == null && !f.mRestored) {
	                                    throwException(new IllegalArgumentException(
	                                            "No view found for id 0x"
	                                            + Integer.toHexString(f.mContainerId) + " ("
	                                            + f.getResources().getResourceName(f.mContainerId)
	                                            + ") for fragment " + f));
	                                }
	                            }
	                            f.mContainer = container;
	                            f.mView = f.performCreateView(f.getLayoutInflater(
	                                    f.mSavedFragmentState), container, f.mSavedFragmentState);
	                            if (f.mView != null) {
	                                f.mView.setSaveFromParentEnabled(false);
	                                if (container != null) {
	                                    Animator anim = loadAnimator(f, transit, true,
	                                            transitionStyle);
	                                    if (anim != null) {
	                                        anim.setTarget(f.mView);
	                                        anim.start();
	                                    }
	                                    container.addView(f.mView);
	                                }
	                                if (f.mHidden) f.mView.setVisibility(View.GONE);
	                                f.onViewCreated(f.mView, f.mSavedFragmentState);
	                            }
	                        }
	
	                        f.performActivityCreated(f.mSavedFragmentState);
	                        if (f.mView != null) {
	                            f.restoreViewState(f.mSavedFragmentState);
	                        }
	                        f.mSavedFragmentState = null;
	                    }
	                case Fragment.ACTIVITY_CREATED:
	                case Fragment.STOPPED:
	                    if (newState > Fragment.STOPPED) {
	                        if (DEBUG) Log.v(TAG, "moveto STARTED: " + f);
	                        f.performStart();
	                    }
	                case Fragment.STARTED:
	                    if (newState > Fragment.STARTED) {
	                        if (DEBUG) Log.v(TAG, "moveto RESUMED: " + f);
	                        f.mResumed = true;
	                        f.performResume();
	                        // Get rid of this in case we saved it and never needed it.
	                        f.mSavedFragmentState = null;
	                        f.mSavedViewState = null;
	                    }
	            }
	        } else if (f.mState > newState) {
	            switch (f.mState) {
	                case Fragment.RESUMED:
	                    if (newState < Fragment.RESUMED) {
	                        if (DEBUG) Log.v(TAG, "movefrom RESUMED: " + f);
	                        f.performPause();
	                        f.mResumed = false;
	                    }
	                case Fragment.STARTED:
	                    if (newState < Fragment.STARTED) {
	                        if (DEBUG) Log.v(TAG, "movefrom STARTED: " + f);
	                        f.performStop();
	                    }
	                case Fragment.STOPPED:
	                case Fragment.ACTIVITY_CREATED:
	                    if (newState < Fragment.ACTIVITY_CREATED) {
	                        if (DEBUG) Log.v(TAG, "movefrom ACTIVITY_CREATED: " + f);
	                        if (f.mView != null) {
	                            // Need to save the current view state if not
	                            // done already.
	                            if (!mActivity.isFinishing() && f.mSavedViewState == null) {
	                                saveFragmentViewState(f);
	                            }
	                        }
	                        f.performDestroyView();
	                        if (f.mView != null && f.mContainer != null) {
	                            Animator anim = null;
	                            if (mCurState > Fragment.INITIALIZING && !mDestroyed) {
	                                anim = loadAnimator(f, transit, false,
	                                        transitionStyle);
	                            }
	                            if (anim != null) {
	                                final ViewGroup container = f.mContainer;
	                                final View view = f.mView;
	                                final Fragment fragment = f;
	                                container.startViewTransition(view);
	                                f.mAnimatingAway = anim;
	                                f.mStateAfterAnimating = newState;
	                                anim.addListener(new AnimatorListenerAdapter() {
	                                    @Override
	                                    public void onAnimationEnd(Animator anim) {
	                                        container.endViewTransition(view);
	                                        if (fragment.mAnimatingAway != null) {
	                                            fragment.mAnimatingAway = null;
	                                            moveToState(fragment, fragment.mStateAfterAnimating,
	                                                    0, 0, false);
	                                        }
	                                    }
	                                });
	                                anim.setTarget(f.mView);
	                                anim.start();
	
	                            }
	                            f.mContainer.removeView(f.mView);
	                        }
	                        f.mContainer = null;
	                        f.mView = null;
	                    }
	                case Fragment.CREATED:
	                    if (newState < Fragment.CREATED) {
	                        if (mDestroyed) {
	                            if (f.mAnimatingAway != null) {
	                                // The fragment's containing activity is
	                                // being destroyed, but this fragment is
	                                // currently animating away.  Stop the
	                                // animation right now -- it is not needed,
	                                // and we can't wait any more on destroying
	                                // the fragment.
	                                Animator anim = f.mAnimatingAway;
	                                f.mAnimatingAway = null;
	                                anim.cancel();
	                            }
	                        }
	                        if (f.mAnimatingAway != null) {
	                            // We are waiting for the fragment's view to finish
	                            // animating away.  Just make a note of the state
	                            // the fragment now should move to once the animation
	                            // is done.
	                            f.mStateAfterAnimating = newState;
	                            newState = Fragment.CREATED;
	                        } else {
	                            if (DEBUG) Log.v(TAG, "movefrom CREATED: " + f);
	                            if (!f.mRetaining) {
	                                f.performDestroy();
	                            }
	
	                            f.mCalled = false;
	                            f.onDetach();
	                            if (!f.mCalled) {
	                                throw new SuperNotCalledException("Fragment " + f
	                                        + " did not call through to super.onDetach()");
	                            }
	                            if (!keepActive) {
	                                if (!f.mRetaining) {
	                                    makeInactive(f);
	                                } else {
	                                    f.mActivity = null;
	                                    f.mParentFragment = null;
	                                    f.mFragmentManager = null;
	                                }
	                            }
	                        }
	                    }
	            }
	        }
	        
	        f.mState = newState;
	    }