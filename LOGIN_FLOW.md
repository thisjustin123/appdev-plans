### This is the breakdown for Eatery's login flows, fully MVVM'd.

# 1. VM $\rightarrow$ UI (Est. Time: 1-2 weeks)

The first and most straightforward (but still lengthy!) part of the MVVM architecture is the interaction between the **viewmodel** and its corresponding **UI** components.

This is generally broken down into two subtasks, in which you are acting as both the client and the implementer (2110!):

1. Design the Viewmodel to send (exclusively through `Flow`s and sometimes through `MutableState`s) all the information down to the UI that it needs to display all dynamic information.

2. Acting now as the client of your own Viewmodel code, use those `Flow`s to show the correct data whenever a packet of data is sent down the `Flow`. Hook them up to your UI components with `.collectAsState()`.

Here's my idea, specifically pertaining to Eatery's Profile:

## 1.1. Use a class like the following: 
```kotlin
sealed class ProfileState {
        data class Login(
          val netid: String, 
          val password: String, 
          val failureMessage: String?,
          val loading: Boolean
        ) : State()

        data class Account(
            val user: User, // Contains all user data.
            var query: String, // Search bar query.
            var accountFilter: AccountType // Search bar filter.
        ) : State()
    }
```
(Don't *blindly* copy this! Tweak things around to see what's best.)

And have a `MutableStateFlow` (encapsulated with a `StateFlow`) to send this information down to some arbitrary UI:

```kotlin
private var _state = MutableStateFlow<ProfileState>(State.Empty)
val state = _state.asStateFlow()
```

Here, `state` is a `StateFlow<ProfileState>` that the UI can read from to decide what information to show.

## 1.2. Write the UI, using your VM code.

Use `state`. Read its flow value with `.collectAsState()`, changing the UI shown as follows:

- When the UI sees a `Login` object, it should show the netid and password typed in. If `loading` is true, make the username/password unchangeable, and show the login button spinning. If there's a non-null error message, show it in a red box (if that's something design has specified).

- When the UI sees an `Account` object, it should show the user profile data loaded from backend. The `query` and `accountFilter` can be used to tell the screen what the user has typed into the search bar and what filters they've applied.

A lot of the UI components here should be already done. (They were working when I was on Eatery. I should know; I made a lot of them.)

Congrats! That's the first part of the MVVM paradigm done.

# 2. Gearing up for User Input (Est Time: 1 week)

Obviously, an app would suck balls if it couldn't take in any user input. A sanity check though until this point is that you really shouldn't have implemented any user input at all. 

Technically, if you obeyed my steps strictly up until this point, you shouldn't even be able to type your netID and password in. Sure, given a netID & password from the VM, you can display it, but typing in shouldn't do anything yet.

Communication from the UI to the VM is the next step here. Your main tasks are:

- Create placeholder functions for all meaningful user inputs (i.e. `onLoginPressed()`, `onNetIDTyped()`) in the VM.

- Call these functions wherever applicable in the UI.

- Implement anything you can right now. That should mean anything not related to backend/model stuff.

Here's an example implementation (possibly buggy) of `onNetIDTyped()`:

```kotlin
fun onNetIDTyped(newNetid : String) {
  val currState = _state.value
  if (currState !is State.Login) return

  // currState is a Login state (expected).
  val loginState = currState as State.Login

  val newState = State.Login(
    newNetid, 
    loginState.password, 
    loginState.failureMessage, 
    false // Should never be able to type in when loading, anyways.
  )

  // Send the new netID Login state down.
  _state.value = newState
}
```

This function essentially reads the previous `Login` info, makes a new state with the new netID the user has typed in, then sends that down the flow. You'd want a similar function for the password, and then call these when the user types a new character in for the netid/password.

For now, you shouldn't know how to implement something like `onLoginPressed()`. Maybe for now, at least just for testing, you can set `loading` to true.

### DID YOU KNOW:

ChatGPT suggests:
```
To make your code cleaner and more efficient, you can use the copy() function provided by data classes in Kotlin. The copy function allows you to create a new instance of the data class with some of its properties modified while keeping the other properties unchanged.
```

Example:
```kotlin
val updatedLoginState = currentLoginState.copy(
    loading = true,
)
```

Thanks, ChatGPT! That cleans up some lines of code. You'd then send the `updatedLoginState` down the flow with `_state.value = updatedLoginState`.

# 3. Model (Est Time: I'm gonna KMS)

Bruh
