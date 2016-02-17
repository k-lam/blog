	ResolveInfo rInfo =
                AppGlobals.getPackageManager().resolveIntent(
                        intent, resolvedType,
                        PackageManager.MATCH_DEFAULT_ONLY
                                    | ActivityManagerService.STOCK_PM_FLAGS, userId);



PackageManagerService.java中

	@Override
    public ResolveInfo resolveIntent(Intent intent, String resolvedType,
            int flags, int userId) {
        if (!sUserManager.exists(userId)) return null;
        enforceCrossUserPermission(Binder.getCallingUid(), userId, false, "resolve intent");
        List<ResolveInfo> query = queryIntentActivities(intent, resolvedType, flags, userId);
        return chooseBestActivity(intent, resolvedType, flags, query, userId);
    }
	 private ResolveInfo chooseBestActivity(Intent intent, String resolvedType,
            int flags, List<ResolveInfo> query, int userId) {
        if (query != null) {
            final int N = query.size();
            if (N == 1) {
                return query.get(0);
            } else if (N > 1) {
                // If there is more than one activity with the same priority,
                // then let the user decide between them.
                ResolveInfo r0 = query.get(0);
                ResolveInfo r1 = query.get(1);
                if (DEBUG_INTENT_MATCHING) {
                    Log.d(TAG, r0.activityInfo.name + "=" + r0.priority + " vs "
                            + r1.activityInfo.name + "=" + r1.priority);
                }
                // If the first activity has a higher priority, or a different
                // default, then it is always desireable to pick it.
                if (r0.priority != r1.priority
                        || r0.preferredOrder != r1.preferredOrder
                        || r0.isDefault != r1.isDefault) {
                    return query.get(0);
                }
                // If we have saved a preference for a preferred activity for
                // this Intent, use that.
                ResolveInfo ri = findPreferredActivity(intent, resolvedType,
                        flags, query, r0.priority, userId);
                if (ri != null) {
                    return ri;
                }
                if (userId != 0) {
                    ri = new ResolveInfo(mResolveInfo);
                    ri.activityInfo = new ActivityInfo(ri.activityInfo);
                    ri.activityInfo.applicationInfo = new ApplicationInfo(
                            ri.activityInfo.applicationInfo);
                    ri.activityInfo.applicationInfo.uid = UserHandle.getUid(userId,
                            UserHandle.getAppId(ri.activityInfo.applicationInfo.uid));
                    return ri;
                }
                return mResolveInfo;
            }
        }
        return null;
    }

对ResolveInfo的解释：
Information that is returned from resolving an intent against an IntentFilter. This partially corresponds to information collected from the AndroidManifest.xml's <intent> tags.

所以

	ResolveInfo rInfo =
                AppGlobals.getPackageManager().resolveIntent(
                        intent, resolvedType,
                        PackageManager.MATCH_DEFAULT_ONLY
                                    | ActivityManagerService.STOCK_PM_FLAGS, userId);
这一句代码的意义在于，根据Intent找到一个最好的（chooseBestActivity）。
