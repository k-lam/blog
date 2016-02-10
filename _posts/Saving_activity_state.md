#[Saving Activity State](http://developer.android.com/guide/components/activities.html#SavingActivityState)
##Brief
为什么会有？因为Activity处于onPause，onStop，onDestroy都可能由于系统内存紧张而被回收，但是这个对用户来说是透明的，如果用户回到这个Activity，发现之前的交互都没有了，这样的用户体验就不好。所以我们的程序需要保持这些可能被回收的状态，如果Activity被回收，当用户再次打开这个Activity时，能恢复用户之前看到的界面。

在Activity中，The system calls onSaveInstanceState() before making the activity vulnerable to destruction.
所以，在onSaveInstanceState()这个方法中进行保存操作。

the system recreates the activity and passes the Bundle to both onCreate() and onRestoreInstanceState()

在onCreate()和onRestoreInstanceState()中恢复

`Activity State：交互界面的状态，譬如输入了什么内容，列表下拉到了哪里。`


##SaveInstanceState

####Activity.onSaveInstanceState

    protected void onSaveInstanceState(Bundle outState) {
        outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
        Parcelable p = mFragments.saveAllState();
        if (p != null) {
            outState.putParcelable(FRAGMENTS_TAG, p);
        }
		//这句不用理，是给用户用的回调，不是系统用的
        getApplication().dispatchActivitySaveInstanceState(this, outState);
    }
可以看到，分成Fragment和Window的saveState。

Fragment的SaveState内容蛮多的，除了各个Fragment的save，还有FragmentManagement的save state。留待以后分析。

mWindow是Window类型，手机的实现类型是PhoneWindow

####PhoneWindowsaveHierarchyState
	static final String FRAGMENTS_TAG = "android:fragments";
	.
	.
	.
    static private final String FOCUSED_ID_TAG = "android:focusedViewId";
    static private final String VIEWS_TAG = "android:views";
    static private final String PANELS_TAG = "android:Panels";
    static private final String ACTION_BAR_TAG = "android:ActionBar";
	public Bundle saveHierarchyState() {
        Bundle outState = new Bundle();
        if (mContentParent == null) {
            return outState;
        }
		
		//save the view
        SparseArray<Parcelable> states = new SparseArray<Parcelable>();
        mContentParent.saveHierarchyState(states);
        outState.putSparseParcelableArray(VIEWS_TAG, states);

        // save the focused view id
        View focusedView = mContentParent.findFocus();
        if (focusedView != null) {
            if (focusedView.getId() != View.NO_ID) {
                outState.putInt(FOCUSED_ID_TAG, focusedView.getId());
            } else {
                if (false) {
                    Log.d(TAG, "couldn't save which view has focus because the focused view "
                            + focusedView + " has no id.");
                }
            }
        }

        // save the panels
        SparseArray<Parcelable> panelStates = new SparseArray<Parcelable>();
        savePanelState(panelStates);
        if (panelStates.size() > 0) {
            outState.putSparseParcelableArray(PANELS_TAG, panelStates);
        }

        if (mActionBar != null) {
            SparseArray<Parcelable> actionBarStates = new SparseArray<Parcelable>();
            mActionBar.saveHierarchyState(actionBarStates);
            outState.putSparseParcelableArray(ACTION_BAR_TAG, actionBarStates);
        }

        return outState;
    }

可以看到，保存view、focused view id、panels和ActionBar。
而加上之前的Fragment的保存，可以很清楚看到Bundle最高层键值格式：

`android:fragments`:value

`android:focusedViewId`:value

`android:views`:value

`android:Panels`:value

`android:ActionBar`:value

###View的saveInstance
mContentParent.saveHierarchyState其实是调用了View.saveHierarchyState->dispatchSaveInstanceState...等等，不对。mContentParent从名字看就知道不是一般的ViewGroup。于是看看mContentParent这个的定义。

	// This is the view in which the window contents are placed. It is either
    // mDecor itself, or a child of mDecor where the contents go.
    private ViewGroup mContentParent;
嗯，mDecor itself,or a child of mDecor wher the contents go.这样，我们就应该看看DecorView，FrameLayout（几大布局选这个而已，当然可以挑其他的研究），ViewGroup，View的saveHierarchyState。
通过对View-ViewGroup—FrameLayout—DectView这个家族的检查，只在ViewGroup中继承了dispatchSaveInstanceState。！！！

####ViewGroup.dispatchSaveInstanceState

	@Override
    protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
        super.dispatchSaveInstanceState(container);
        final int count = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < count; i++) {
            View c = children[i];
            if ((c.mViewFlags & PARENT_SAVE_DISABLED_MASK) != PARENT_SAVE_DISABLED) {
                c.dispatchSaveInstanceState(container);
            }
        }
    }

这里就很明显了，调用children（View）的dispatchSaveInstanceState。

####View.dispatchSaveInstanceState

    protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
        if (mID != NO_ID && (mViewFlags & SAVE_DISABLED_MASK) == 0) {
            mPrivateFlags &= ~PFLAG_SAVE_STATE_CALLED;
            Parcelable state = onSaveInstanceState();
            if ((mPrivateFlags & PFLAG_SAVE_STATE_CALLED) == 0) {
                throw new IllegalStateException(
                        "Derived class did not call super.onSaveInstanceState()");
            }
            if (state != null) {
                // Log.i("View", "Freezing #" + Integer.toHexString(mID)
                // + ": " + state);
                container.put(mID, state);
            }
        }
    }

嗯，应了官网说的，没有android:id的不会save，if中 mID != NO_ID可以看出来

####View.onSaveInstanceState

    protected Parcelable onSaveInstanceState() {
        mPrivateFlags |= PFLAG_SAVE_STATE_CALLED;
        return BaseSavedState.EMPTY_STATE;
    }

EMPTY_STATE,也就是说，各个View要自己Override onSaveInstanceState，决定保存什么内容了。

###Panel

#####PhoneWindow.savePanelState()

    private void savePanelState(SparseArray<Parcelable> icicles) {
        PanelFeatureState[] panels = mPanels;
        if (panels == null) {
            return;
        }

        for (int curFeatureId = panels.length - 1; curFeatureId >= 0; curFeatureId--) {
            if (panels[curFeatureId] != null) {
                icicles.put(curFeatureId, panels[curFeatureId].onSaveInstanceState());
            }
        }
    }

`由于暂时不知道panel是什么，所以这部分to be continued...`

###ActionBar

ActionBar兜兜转转，最后其实还是调用ViewGroup的dispatchSaveInstanceState,参考上文吧

##RestoreInstanceState

restore在Activity中有两个入口`onCreate`和`onRestoreInstanceState`

###onCreate

    protected void onCreate(Bundle savedInstanceState) {
	   	......
        if (savedInstanceState != null) {
            Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
            mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                    ? mLastNonConfigurationInstances.fragments : null);
        }
        ......
    }

可以看到，onCreate中主要是恢复Fragment的State的，Fragment的内容后续讨论。

###onRestoreInstanceState

    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        if (mWindow != null) {
            Bundle windowState = savedInstanceState.getBundle(WINDOW_HIERARCHY_TAG);
            if (windowState != null) {
                mWindow.restoreHierarchyState(windowState);
            }
        }
    }
mWindowState如果是手机，就是PhoneWindow类型

####PhoneWindow.restoreHierarchyState

    public void restoreHierarchyState(Bundle savedInstanceState) {
        if (mContentParent == null) {
            return;
        }

        SparseArray<Parcelable> savedStates
                = savedInstanceState.getSparseParcelableArray(VIEWS_TAG);
        if (savedStates != null) {
            mContentParent.restoreHierarchyState(savedStates);
        }

        // restore the focused view
        int focusedViewId = savedInstanceState.getInt(FOCUSED_ID_TAG, View.NO_ID);
        if (focusedViewId != View.NO_ID) {
            View needsFocus = mContentParent.findViewById(focusedViewId);
            if (needsFocus != null) {
                needsFocus.requestFocus();
            } else {
                Log.w(TAG,
                        "Previously focused view reported id " + focusedViewId
                                + " during save, but can't be found during restore.");
            }
        }

        // restore the panels
        SparseArray<Parcelable> panelStates = savedInstanceState.getSparseParcelableArray(PANELS_TAG);
        if (panelStates != null) {
            restorePanelState(panelStates);
        }

        if (mActionBar != null) {
            SparseArray<Parcelable> actionBarStates =
                    savedInstanceState.getSparseParcelableArray(ACTION_BAR_TAG);
            if (actionBarStates != null) {
                mActionBar.restoreHierarchyState(actionBarStates);
            } else {
                Log.w(TAG, "Missing saved instance states for action bar views! " +
                        "State will not be restored.");
            }
        }
    }

对应了onSaveInstance中，save View,focused view,Panels,ActionBar.
在restore的时候，也是restore View,focused view,Panels,ActionBar.

我们这里可以大胆猜想，restore的过程，应该和save一样。下面就来验证，如果是一样的，就不赘述。只指出不同的地方。

检验中...

......

结果发现View和ActionBar是一样的。也就是

ViewGroup去dispath，调用View的dispatch，view的dispatch调用view的onRestoreInstanceState，view就可以在这里恢复了。




