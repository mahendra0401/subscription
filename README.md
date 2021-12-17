# Subscription App

Instead of the default functionality of Open edX to provide course access to learners, make it subscription based

## App Configuration
* Add `subscription` django app at `edx-platform/common/djangoapps` directory
* Add `custom-theme` at `edx-platform/themes` directory
* Ensure comprehesive theme enabled. Use `custom-theme`
* Add `common.djangoapps.subscription.apps.SubscriptionConfig` to the INSTALLED_APPS (i.e. in `edx-platform/lms/envs/common.py`)
* Add to lms url (i.e. in `edx-platform/lms/urls.py`): 
```python
urlpatterns += [
    url(r'^', include('common.djangoapps.subscription.urls')),
]
```
* Add into the settings file (i.e. in `edx-platform/lms/envs/common.py`): 
```python
REGISTRATION_EXTENSION_FORM = "common.djangoapps.subscription.forms.UserInfoForm"
```
* Apply migration
* Add below changes into the (`lms/djangoapps/courseware/access.py`) inside `def can_load():`: 
```python
def _has_access_course(user, action, courselike):
    @function_trace('can_load')
    def can_load():
        ...
        ...
        ...
        # Subscription Related changes
        from common.djangoapps.subscription.access import _has_subscription_access
        subscription_access = _has_subscription_access(user)
        if not subscription_access:
            staff_access = _has_staff_access_to_descriptor(user, courselike, courselike.id)
            if staff_access:
                return staff_access
            else:
                return subscription_access

        return ACCESS_GRANTED

    @function_trace('can_enroll')
    def can_enroll():
        """
        Returns whether the user can enroll in the course.
        """
        return _can_enroll_courselike(user, courselike)
```
* Restart your Open edX processes(lms and cms/studio)
* Setup and run daily cronjob using below command to update subscription: 
```python
./manage.py lms check_subscription --settings=production    - for production setup
./manage.py lms check_subscription --settings=devstack_docker    - for devstack setup
```
## Use Subscription
* Admin can create subscription from admin panel `/admin/subscription/subscription/`
* Admin can see all activated subscriptions from admin panel `/admin/subscription/usersubscription/`
* Users can see all subscription at `/subscriptions`
