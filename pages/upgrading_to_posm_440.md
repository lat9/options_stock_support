# Updating Products Options Stock Manager to v4.4.0

2022 brought us Zen Cart v1.5.8 and _POSM_ v4.4.0 provides that integration.  If you've purchased an earlier version of _POSM_, this document identifies the changes you might need to make in your update to v4.4.0.  The major changes in this version are:

1. [Notification Changes](./posm_notifications.md).  *POSM's* notifications have been modified to use a `NOTIFY_` prefix &mdash; otherwise, any observers that use the [camelCased event-name syntax](https://docs.zen-cart.com/dev/code/notifiers/#camelcased-event-name) will  fail to get that message in zc158!  The *legacy* notifications are still present in *POSM* v4.4.0 to support any back-testing needed to your custom observers as you convert them to use the now-current forms of notification. ***Note***: All *legacy* notifications will be removed in a future version of *POSM*.
2. *POSM*'s admin tools now use the Bootstrap framework introduced in the Zen Cart 1.5.6 admin.
3. Updates to support interoperation with PHP 8.2.

