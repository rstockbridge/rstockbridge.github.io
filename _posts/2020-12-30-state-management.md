---
layout: post
title: Refactoring for State Management
comments: true
---

Remember when forcing an app to portrait mode prevented (most) configuration changes? And so it was reasonable to ignore the tedious task of preserving application state after a configuration change? Ahh those were the days...

The release of dark mode in Android 10 in 2019 forced many of us to think much more carefully about state management. Toggling between light and dark mode induces a configuration change, and users can switch anywhere in any app flow. It's no longer enough to lock to portrait mode and choose to not handle the other, usually infrequent, cases such as system-initiated process death.

In anticipation of dark mode, I spent several months last year updating a legacy app to handle state management properly. In this post, I will share our strategies to produce an excellent user experience while accepting the realities of an app that was not written with state management in mind. So much work was necessary because the app uses MVP architecture and it was impractical to re-architect to MVVM. A more modern app that issues complete view states bound to the view layer would be in much better shape to begin with.

Before getting into the details, I want to give full credit to Cory Gorham, who led this initiative and proposed most of these approaches. He kept our spirits up as we reconsidered and reworked every part of the app. Thanks Cory!

## Configuration Changes

First, a quick recap of [configuration changes](https://developer.android.com/guide/topics/resources/runtime-changes) on Android. 

When a configuration change is triggered, the current activity will be destroyed and recreated. This allows the activity to update itself correctly for the new configuration. 

The most common example is switching your device between portrait and landscape orientation. Other examples include changing font scaling and updating display density. [This page](https://developer.android.com/guide/topics/manifest/activity-element#config) lists even more situations, most of them uncommon.

You may also wish to preserve state after forced process death by the system to free up memory.

You as the developer are responsible for making sure all necessary state is saved to be able to recreate the activity exactly as it was before. This is not as bad as it sounds! The Android system takes care of restoring the state of views in your activity as long as they have a unique ID (this is more useful for MVP than for MVVM). And ViewModels will automatically retain data across configuration changes. However, ViewModels do not survive process death, so explicit state management is required if you want to handle process death gracefully. 

That's all for this brief overview. For a deeper dive, I refer you to [this article](https://developer.android.com/topic/libraries/architecture/saving-states), which covers all the options for saving state in detail.

## Strategies

As mentioned above, the app I worked on is many years old with an entrenched MVP architecture. We felt it would be too difficult and risky to change the entire architecture and so decided to work around it instead. Many of the strategies I'll discuss below are a result of that limitation, and may not be relevant to your particular project. Hopefully you'll still find one or two useful tips!

We started the process for doing this work by first meeting as a team and listing out our strategies. A major part of the discussion was what behavior was an absolute requirement, and what could we compromise on slightly for the sake of time (see #10). We then went through every screen of the app one-by-one and made changes as necessary. Verifying state has been restored correctly can be more subtle than just checking the right data is appearing on the screen (see #3) and so each screen was tested very thoroughly for any new issues. Of course we also made new discoveries along the way. 

Here are the strategies we used:

**1. Clean up architecture if necessary**

[Everything is harder](https://www.rstockbridge.dev/2020-07-20-architecture-consistency/) if your chosen architecture pattern is not implemented consistently. This app still had a few rough areas that didn't follow MVP conventions, so we took this as an opportunity to clean things up. It was then much easier to identify what aspects of state management were necessary to focus on for each screen.

**2. Centralize maintaining the state of each view**

This is related to #1, and probably not much of a concern for apps following the most current architectural guidelines. But I have encountered situations where screen state is affected by code across multiple files, and it's very difficult to predict the overall state of the app at any given time because you have to look in so many places.

If you know where your state is coming from, it's much easier to make sure it's maintained.

**3. Think about all state, not just what's visible**

It's tempting to verify that a screen handles dark mode properly by toggling dark mode and confirming that the screen looks the same. That is not sufficient - any state living outside views needs to be preserved as well. 

For example, maybe you have a variable that stores the most recent value received from a network request, and the behavior of the screen depends on if the next network request returns the same value or a new value. You'll need to save the value of this variable so that it re-initializes appropriately.

If necessary, you can also add new properties to explicitly track screen-wide state changes.

**4. Use `onSaveinstanceState()` and `onRestoreInstanceState()`**

Since we didn't have ViewModels available to us, we needed to use these standard methods to save all data that wasn't already being saved automatically.

**5. Make sure that view setup logic happens after state restoration**

You want to make sure you use your restored state.

**6. Pay attention to asynchronous actions that start before state restoration**

Examine what will happen to that asynchronous action after a configuraton change. 

- Will the action be terminated or will it continue?
- Will the appropriate listener still be listening for the result?
- Will be action be triggered again automatically when the screen is recreated (e.g., it happens in `onStart()` or `onResume()`). Do you *want* the action to happen again?

If a network request involves fetching data, it may be no issue at all to simply make the request again. You may also be able to move that request to a different part of the app flow.

Requests that push data require more thought and care. For example, maybe your user adds an item to their cart and then rotates their phone. You almost certainly don't want to rerun that request and duplicate the item. 

**7. Save state of custom views** 

Custom views require additional code to save their own state. It can be a little fiddly and confusing; [this article](https://www.netguru.com/codestories/how-to-correctly-save-the-state-of-a-custom-view-in-android) does a great job of explaining what to do.

**8. Pay attention to fragment setup and state**

Fragments may depend on setup happening in their parent fragments or activities. During state restoration, the order in which fragment lifecycle methods are called may depend on when lifecycle methods are called in their parent fragment or activity.

**9. Pay attention to the back stack**

All activities and fragments are re-created after a configuration change, not just what's visible. I once spent a very, very long time trying to track down the source of a bug by focusing on the recreation steps and flow, just to finally determine the problem was in a hidden activity.

**10. Save time by only persisting essential dialogs**

A key difference between Dialogs and DialogFragments is that DialogFragments are automatically re-created upon a configuration change. If you're making a new dialog, I recommend using a DialogFragment.

However, we were faced with an app full of Dialogs, and it would have taken a lot of time to convert them all to DialogFragments. We decided to only convert dialogs that could not be easily regenerated by the user. If the user could pop the dialog back up by pressing a button, we did not worry about making sure it persisted.

## Conclusion

While evaluating and updating an entire app for state management was quite an undertaking, I actually enjoyed the challenge of creating a good user experience given the constraints of working with a legacy codebase. We really honed in on what would provide value, rolled up our sleeves, and jumped in. Plus this was a great chance to think deeply about app state, app flows, and lifecycle. However, I won't mind working with a more modern setup in the future! 