### MVI with Jetpack Compose

**Main Idea: Make View lean**

- What Jetpack looks like (briefly)

- Usual way of MVI
  ViewModel get data from the data source
  View observe the state of ViewModel

  ```kotlin
  // View
  fun observeState() {
      vm.state.observe(this) { state ->
          when (state) {
              Loading -> {}
              Error -> {}
              Success -> {}
          }
      }
  }
  ```

  Problem: Why View check the state?

- Solution
  ViewState build the UI
  View renders ViewState

  ```kotlin
  // ViewState
  sealed class ViewState {
      @Composable
      abstract fun buildUi()
      
      object Loading : ViewState() {
          @Composable
          override fun buildUi() {
              Padding(8.dp) {
                  ...
              }
          }
      }
  }
  ```

  > View should represent the View, so it should know how to render it

- Downsides - No XML, No View hierarchy
  Do we care? NO
  -> We don't need them
  -> Because we know how it is built.

### Diving into Jetpack Compose