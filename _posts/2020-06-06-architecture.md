---
layout: post
title: Architecture Trio Of Apps
comments: true
---

Last year, I spent some time diving into common architecture patterns to learn how they apply to Android apps. Specifically, I focused on Model-View-Controller (**MVC**), Model-View-Presenter (**MVP**), and Model-View-ViewModel (**MVVM**). I quickly became a bit overwhelmed.

Since architecture patterns are independent of software and platform or language and framework, there are many definitions and implementations in use. I read many helpful tutorials but I found it challenging to take a step back and understand the big picture. **What are the goals of each pattern? How do they affect what behavior or state can be tested? What are the pros and cons when it comes to their use on Android?**

The typical arrow diagrams demonstrating the relationships between entities in each pattern also didn't fully capture all the relationships I was interested in (i.e., which classes hold a reference to which classes). This is particularly important for writing tests.

The purpose of this post is to fill in those gaps. I will describe three nearly identical apps, differing only in **architecture** and **testing strategy**. The apps are as simple as possible while introducing just enough complexity to capture the architectural differences.

## Greeting App

Each app displays a series of greetings in a loop ("Hi!", "Hello!", "Welcome!"). The next greeting displays when the user clicks the button.

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/app.gif" width="30%" /></div>

It's important to include dynamic UI and basic user interaction to draw out key variations in the architectures.

## MVC

The main takeways of the MVC pattern on Android are:

- **Fat activity:** this is where all the action takes place. <br>
- **Difficult to write tests:** minimal separation of concerns between business logic and UI.

The diagram below shows a typical implementation of MVC. `GreetingActivity` acts as both the view and the controller, while `GreetingModel` is the model. `GreetingActivity` depends on the model (as well as an Android framework class with complex lifecycle), as shown at the bottom of the square in light blue, while `GreetingModel` has no dependencies. When the user clicks the button, `GreetingActivity` passes the click index (count) to `GreetingModel` and `GreetingModel` returns the greeting for that index.

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/mvc_diagram.png" width="70%" /></div>

I implemented this pattern in the app [GreetingMVC](https://github.com/rstockbridge/GreetingMVC). For the project structure, I created two packages, `model` and `controller`, to really emphasize which class plays which role.

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/mvc_file_structure.png" width="50%" /></div>

Here is the **model** class. 
This class simply returns a greeting based on the index that is passed in.

```kotlin
class Model {
    private val messages = arrayOf("Hi!", "Hello!", "Welcome!")

    fun getMessage(index: Int): String {
        return messages[index % messages.size]
    }
}
```

Next up is the **view/controller** class, which is an `Activity`. You can see the dependency on `GreetingModel`, as well as the logic to respond to a button click by showing a message fetched from `GreetingModel`. 

```kotlin
class MainActivity : AppCompatActivity() {
    ...
    private val model = Model()
    private var clickCount = 0

    override fun onCreate( ... ) {
        ...
        binding.button.setOnClickListener {
            clickCount++
            showMessage(model.getMessage(clickCount))
        }

        showMessage(model.getMessage(_c))
    }
    
    private fun showMessage(text: String) {
        binding.label.text = text
    }
}
```

Unfortunately, it is not possible to write simple unit tests for `GreetingMVC`. We are limited to creating more complicated UI tests since the business logic (responding to a click with a new message) is part of an `Activity`. (The dependency on the Android framework in the diagram is a warning sign that this will be the case!) While we have begun to implement separation of concerns by separating out the exact selection of the message in `GreetingModel`, we haven't gone far enough.

Note that the UI state also doesn't survive configuration changes without additional work.

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/not_rotation_safe.gif" width="30%" /></div>

I also came across the following variations:

- The `Activity` is the controller and the XML is the view. This approach seems to be more about renaming classes than changing the underlying structure, so it will have the same strengths and weaknesses as our example. <br>
- The view is extracted to be outside the `Activity`. One way to do this would be using custom views, but they are still reliant on the Android framework and lifecycle. <br>

To wrap up MVC, it begins to address separation of concerns by splitting the model from the view, which is better than all logic being in the activity. However, we can't easily test the controller (activity) logic and we have to manually save state to handle configuration changes. We'll address the first concern in the next architecture pattern.

## MVP

The main takeways of the MVP pattern on Android are:

- **Thin activity:** just handles UI. <br>
- **Fat presenter:** the brain. <br>
- Test the presenter's **interaction with the view**.

The common implementation of MVP is summarized by this diagram. `GreetingModel` remains the same as for MVC, but the UI and business logic responsibilities are now split between `GreetingActivity` (the view) and `GreetingPresenter` (the presenter). `GreetingActivity` simply notifies `GreetingPresenter` when the user clicks the button, and displays the message when the `GreetingPresenter` tells it to do so. It must hold a reference to `GreetingPresenter` to perform its notification duties. `GreetingPresenter` is the brains of the operation - it decides what to do when notified about a button click (fetch the appropriate message and tell `GreetingActivity` to display it). As such, it holds references to both the view and the model. 

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/mvp_diagram.png" width="100%" /></div>

[GreetingMVP](https://github.com/rstockbridge/GreetingMVP) demonstrates this approach. The project structure contains three packages, `model`, `presenter`, and `view`. Note that the `view` package contains two classes - `GreetingView ` and `GreetingActivity`. `GreetingView` is the interface that facilitates communication between `GreetingActivity` and `GreetingPresenter`. Using an interface facilitates testing the presenter, as we'll see below. 

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/mvp_file_structure.png" width="40%" /></div>

First up is the **view interface** class `GreetingView`. It requires that any implementation of the view must provide a function to display the provided message.

```kotlin
interface GreetingView {
    fun showMessage(text: String)
}
```

Next is the **view** implementation (typically an `Activity` or `GreetingPresenter`). Note its implementation of `GreetingView` and dependencies on `GreetingModel` and `GreetingPresenter`.

```kotlin
class MainActivity : AppCompatActivity(), GreetingView {
    ...
    private val model = Model()
    private lateinit var presenter: Presenter

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        binding.button.setOnClickListener { presenter.onButtonClicked() }

        presenter = Presenter(this, Model())
        presenter.onCreate()
    }
    
    override fun showMessage(text: String) {
        binding.label.text = text
    }
}
```

And finally the most important piece, the **presenter** class. It relies on a `GreetingView` implementation and a `GreetingModel`. The view calls `onButtonClicked()` when appropriate, and `GreetingPresenter` responds by fetching the appropriate message from `GreetingModel` and telling the view to display it via `view.showMessage()`. I did not use an interface to hide away the specific implementation of `onButtonClicked()` in `GreetingPresenter` - I'll explain why soon.

```kotlin
class Presenter(private val view: GreetingView, private val model: Model) {
    private var clickCount = 0

    fun onCreate() {
        view.showMessage(model.getMessage(clickCount))
    }

    fun onButtonClicked() {
        clickCount++
        view.showMessage(model.getMessage(clickCount))
    }
}
```

Now that the business logic and framework/UI are separated, we can write unit tests. We will test the presenter only rather than both the presenter and the view. This is because the view is just responsible for very simple UI tasks, and I trust that the Android widgets are working correctly. 

That's why I didn't bother to hide `GreetingPresenter`'s `onButtonClicked()` function behind an interface - we do not need to be able to mock `GreetingPresenter` because we are not going to test `GreetingActivity`. (Remember in MVP the view holds a reference to the presenter.)

However, to test `GreetingPresenter`, we definitely need to mock `GreetingView`, which you can see in this example **test**. Notice that we are testing how the presenter interacts with the view (asking the view to display a particular message) rather than testing the presenter's intrinsic state (click count).

```kotlin
@Test
fun `One button click shows 2nd message`() {
    // arrange
    val view = mockk<GreetingView>(relaxed = true)
    val model = Model()
    val presenter = Presenter(view, model)

    // act
    presenter.onButtonClicked()

    // assert
    verify { view.showMessage(model.getMessage(1)) }
}
```

To recap, compared to MVC, MVP has an even better separation of concerns, and now we can test the presenter. However, there are some downsides in the context of Android apps:

- Each view (`Activity`) requires its own presenter. This can result in many classes.<br>
- By default, UI state still doesn't survive configuration changes. <br>
- Testing the presenter requires mocking the view, which means hand-writing mocks or adding a new dependency to your application.

Spoiler alert...MVVM will address some of these concerns.

## MVVM

The main takeways of the MVVM pattern on Android are:

- **Thin activity:** just handles UI. <br>
- **Fat view model:** the brain. <br>
- Test the **view model's state**. <br>
- **Less tightly coupled:** view model has no knowledge of the view. 

This diagram illustrates the MVVM pattern. `GreetingModel` continues to be the same as for MVC and MVP. The separation of responsibilities is very similar to MVP, with the `GreetingViewModel` replacing `GreetingPresenter` from MVP. (And so the view, `GreetingActivity`, holds a reference to the view model.) The key difference is that `GreetingViewModel` no longer depends on the view, just on the model (and implicitly on an Android framework class, `ViewModel`, with no lifecycle.) Rather than communicating directly with the view, `GreetingViewModel` relies on a reactive approach using `LiveData` to communicate from the model to the view.

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/mvvm_diagram.png" width="100%" /></div>

[GreetingMVVM](https://github.com/rstockbridge/GreetingMVVM) is the final sample app. The project structure contains three packages, `model`, `presenter`, and `viewmodel`. In contrast to MVP, no interface is required for the view since `GreetingViewModel` no longer depends on the view.

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/mvvm_file_structure.png" width="40%" /></div>

The **view** class, an `Activity`, is below. Notice the use of the observer pattern. `GreetingActivity` listens for any changes in the message value, and displays the new message.

```kotlin
class MainActivity : AppCompatActivity() {
    ...
    private lateinit var viewModel: GreetingViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        binding.button.setOnClickListener { viewModel.onButtonClicked() }

        viewModel = ViewModelProvider(this).get(GreetingViewModel::class.java)
        viewModel.message.observe(this, Observer { message -> showMessage(message) })
    }
}
```

Here is the **view model** class. While the view calls `onButtonClicked()` as in MVP, `GreetingViewModel` relies on `LiveData` to propagate changes in the message value. Note that the mutable internal state is exposed by immutable `LiveData`, so that the view can't accidentally mutate view model state.

```kotlin
class GreetingViewModel : ViewModel() {
    private val model: Model = Model()
    private val clickCount = MutableLiveData(0)
    val message: LiveData<String> = Transformations.map(clickCount, model::getMessage)

    fun onButtonClicked() {
        clickCount.value = cclickCount.value!! + 1
    }
}
```

And finally, here is a **test**. In this case, we are checking the current value of the message in `GreetingViewModel`, i.e., we're testing view model state.

```kotlin
@Test
fun `One button click displays 2nd message`() {
    // arrange 
    val model = Model()
    val viewModel = GreetingViewModel()

    // act
    viewModel.onButtonClicked()

    // assert
    assertEquals(viewModel.message.value, model.getMessage(1))
}
```

MVVM has many advantages over MVC and MVP on Android:

- Survives configuration changes automatically (thank you Architecture Components). This is particularly useful given today's requirements to support Dark Mode.
- Can test view model state. The view model doesn't depend on the view, so there is no need to test how data will be consumed. <br>
- Less-tight coupling. The view can hold references to multiple view models, making it easier to split large pieces of logic into smaller chunkss. <br>

<div style="text-align: center"><img src="/assets/img/2020-06-06-architecture/rotation_safe.gif" width="30%" /></div>

I haven't yet experienced any cons with MVVM, and not surprisingly the Android framework team recommends using MVVM with `ViewModel` in most situations.

---

What have your experiences been with different architectures in Android apps? I'd love to hear your thoughts.