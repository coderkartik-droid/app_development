# Introduction to BLoC in Flutter

## 📖 Introduction

In the world of enterprise Flutter development, one architectural pattern stands as the undisputed industry standard for mission-critical, large-scale applications: **BLoC (Business Logic Component)**.

Originally conceived by Google engineers Paolo Soares and Cong Hui at DartConf 2018 and later revolutionized by Felix Angelov via the official `bloc` and `flutter_bloc` packages, BLoC is not merely a state management tool—it is an **event-driven architectural design pattern**.

The philosophy of BLoC is built on a simple, immutable contract:
1. **Events in**: The user interface does not call methods or change variables directly. It simply reports facts: *"User tapped Login button"* or *"User pulled down to refresh"*. These are **Events**.
2. **States out**: The BLoC processes the event, interacts with repositories, applies business rules, and emits an immutable **State**.
3. **The UI is a pure reflection of the emitted state**: The UI rebuilds to reflect the new state.

By converting all user interactions into a continuous, asynchronous stream of **[Events] $\rightarrow$ [BLoC] $\rightarrow$ [States]**, BLoC provides unmatched predictability, traceability, and testability.

In this chapter, we will master the foundations of the BLoC pattern, inspect how it uses Dart Streams under the hood, explore the core trio of `BlocProvider`, `BlocBuilder`, and `BlocListener`, and build a reactive enterprise authentication pipeline.

---

## 🤔 Why Do We Need It?

Why use BLoC when we already have `ChangeNotifier` and `StateNotifier`?

In standard state management (like Provider):
- A button directly calls a method: `onPressed: () => cart.checkout()`.
- The method can do anything: modify 5 variables, call an API, or trigger a navigation event.
- There is **no standardized audit log** of what events occurred, in what order, or how the state transitioned.

In BLoC:
- **Strict Event-Driven Separation**: The UI cannot call methods. It can *only* dispatch an event: `bloc.add(CheckoutRequested())`.
- **Traceability (Observer Pattern)**: You can install a global `BlocObserver` that logs every single event dispatched and state transitioned across the entire application to your terminal or analytics server!
- **Concurrency Control**: What happens if the user taps "Submit Order" 10 times in 1 second?
  - With standard `setState` or `ChangeNotifier`, 10 simultaneous network requests hit your server!
  - With BLoC, you can specify event transformers (e.g., `droppable()`, `debounce()`, or `restartable()`) to cleanly drop duplicate taps or cancel previous requests in one line of code!

---

## 🌍 Real-Life Analogy

```
┌────────────────────────────────────────────────────────┐
│                   THE VENDING MACHINE                  │
│                                                        │
│  [Events In]:                                          │
│  You stand before a snack vending machine. You cannot  │
│  reach inside and grab the chocolate bar directly.     │
│  You interact strictly via discrete physical events:   │
│  • Event 1: Insert $2 Coin (CoinInserted)              │
│  • Event 2: Press Button "B4" (SelectionPressed)       │
│                                                        │
│  [BLoC Internal Processing]:                           │
│  The internal computer verifies:                       │
│  1. Is $2 enough for item B4? (Business validation)    │
│  2. Is item B4 in stock? (Inventory check)             │
│                                                        │
│  [States Out]:                                         │
│  • State 1: Displaying "Dispensing..." (LoadingState)  │
│  • State 2: Drops chocolate bar into tray (Success)    │
│  • State 3: Beeps and flashes "Out of Stock" (Error)   │
└────────────────────────────────────────────────────────┘
```

A BLoC is a secure vending machine. The outside world can only submit coins and press buttons (Events); the internal mechanism processes the inputs and dispenses the item or displays an error (States).

---

## 🧠 Internal Working: The Stream Pipeline

Under the hood, a BLoC is a **reactive Stream processor**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                          THE BLoC ARCHITECTURE                         │
└────────────────────────────────────────────────────────────────────────┘

    USER GESTURE / UI                      BLoC COMPONENT
  ┌──────────────────┐               ┌────────────────────────┐
  │ User taps Login  │               │ StreamController<Event>│
  └────────┬─────────┘               └───────────┬────────────┘
           │                                     │
           ▼ bloc.add(LoginSubmitted())          ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. Event added to Incoming Event Stream                     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Event Handler: on<LoginSubmitted>((event, emit) async {  │
│    emit(LoginInProgress());                                 │
│    final user = await authRepo.login(...);                  │
│    emit(LoginSuccess(user));                                │
│ })                                                          │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. State emitted to Outgoing State Stream (Stream<State>)   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
  ┌──────────────────────────────────────────────────────────┐
  │ BlocBuilder<AuthBloc, AuthState>                         │
  │ Receives new state -> Rebuilds UI cleanly!               │
  └──────────────────────────────────────────────────────────┘
```

### The Invariant Rules of BLoC:
1. **Events are Inputs**: Data entering the BLoC must be packaged as an Event class.
2. **States are Outputs**: Data leaving the BLoC must be packaged as an immutable State class.
3. **Handlers use `emit()`**: You never mutate state directly; you call the `emit(newState)` callback inside an `on<Event>` registration.

---

## ⚙ Syntax: Anatomy of a Modern BLoC (Bloc 8.0+)

### 1. Events Definition
```dart
abstract class AuthEvent {}

class LoginSubmitted extends AuthEvent {
  final String email;
  final String password;
  LoginSubmitted(this.email, this.password);
}

class LogoutTapped extends AuthEvent {}
```

### 2. States Definition
```dart
abstract class AuthState {}

class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthSuccess extends AuthState {
  final User user;
  AuthSuccess(this.user);
}
class AuthFailure extends AuthState {
  final String error;
  AuthFailure(this.error);
}
```

### 3. The BLoC Implementation
```dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository authRepository;

  AuthBloc(this.authRepository) : super(AuthInitial()) {
    // Register event handler
    on<LoginSubmitted>((event, emit) async {
      emit(AuthLoading());
      try {
        final user = await authRepository.login(event.email, event.password);
        emit(AuthSuccess(user));
      } catch (e) {
        emit(AuthFailure('Invalid credentials'));
      }
    });

    on<LogoutTapped>((event, emit) {
      emit(AuthInitial());
    });
  }
}
```

---

## 📋 The Core Flutter BLoC Widgets

The `flutter_bloc` package provides 4 primary widgets:

| Widget | Purpose | Rebuilds UI? | Where to Use |
| :--- | :--- | :---: | :--- |
| **`BlocProvider`** | Injects the BLoC into the widget tree and manages its lifecycle (auto-dispose). | ❌ No | Near top of tree or route entry. |
| **`BlocBuilder`** | Listens to state changes and rebuilds its UI callback whenever a new state arrives. | **✔ YES** | Returning visual layouts. |
| **`BlocListener`** | Listens to state changes to execute **side effects** (navigation, SnackBars, dialogs). | ❌ No | Showing alerts, routing. |
| **`BlocConsumer`** | Combines `BlocBuilder` and `BlocListener` in a single widget. | **✔ YES** | When a screen needs both visual updates AND dialogs. |

---

## 💻 Complete Runnable Example

The following standalone example demonstrates the **complete BLoC pattern from scratch**:
- Implements a pure framework equivalent of `CustomBloc<Event, State>`, `CustomBlocProvider`, and `CustomBlocBuilder`.
- Implements an **Enterprise ATM Cash Withdrawal Engine**:
  - Events: `InsertCardEvent`, `WithdrawCashEvent(amount)`, `EjectCardEvent`.
  - States: `IdleState`, `CardInsertedState`, `ProcessingState`, `DispensedSuccessState`, `ErrorState`.
- Demonstrates how the UI dispatches events and reacts purely to emitted states!

```dart
import 'dart:async';
import 'package:flutter/material.dart';

void main() {
  runApp(const BlocMasteryApp());
}

// ============================================================================
// 1. ATM EVENTS (Inputs)
// ============================================================================
abstract class AtmEvent {}

class InsertCardEvent extends AtmEvent {
  final String cardNumber;
  InsertCardEvent(this.cardNumber);
}

class WithdrawCashEvent extends AtmEvent {
  final double amount;
  WithdrawCashEvent(this.amount);
}

class EjectCardEvent extends AtmEvent {}

// ============================================================================
// 2. ATM STATES (Outputs)
// ============================================================================
abstract class AtmState {}

class AtmIdleState extends AtmState {}

class AtmCardInsertedState extends AtmState {
  final String maskedCard;
  final double availableBalance;
  AtmCardInsertedState({required this.maskedCard, required this.availableBalance});
}

class AtmProcessingState extends AtmState {
  final String message;
  AtmProcessingState(this.message);
}

class AtmDispensedSuccessState extends AtmState {
  final double amountDispensed;
  final double remainingBalance;
  AtmDispensedSuccessState({required this.amountDispensed, required this.remainingBalance});
}

class AtmErrorState extends AtmState {
  final String errorMessage;
  AtmErrorState(this.errorMessage);
}

// ============================================================================
// 3. PURE DART IMPLEMENTATION OF BLoC ENGINE
// ============================================================================
abstract class CustomBloc<Event, State> {
  CustomBloc(State initialState) : _state = initialState;

  State _state;
  final _stateController = StreamController<State>.broadcast();
  final Map<Type, Function> _handlers = {};

  State get state => _state;
  Stream<State> get stream => _stateController.stream;

  void on<E extends Event>(FutureOr<void> Function(E event, void Function(State state) emit) handler) {
    _handlers[E] = handler;
  }

  void add(Event event) async {
    final handler = _handlers[event.runtimeType];
    if (handler != null) {
      await handler(event, (State newState) {
        _state = newState;
        _stateController.add(_state);
      });
    }
  }

  void dispose() {
    _stateController.close();
  }
}

// ============================================================================
// 4. ATM BLOC (The Business Logic Engine)
// ============================================================================
class AtmBloc extends CustomBloc<AtmEvent, AtmState> {
  double _accountBalance = 850.00;

  AtmBloc() : super(AtmIdleState()) {
    // Handler 1: Insert Card
    on<InsertCardEvent>((event, emit) async {
      emit(AtmProcessingState('Authenticating microchip and PIN...'));
      await Future.delayed(const Duration(milliseconds: 1000));
      emit(AtmCardInsertedState(
        maskedCard: '•••• •••• •••• ${event.cardNumber.substring(event.cardNumber.length - 4)}',
        availableBalance: _accountBalance,
      ));
    });

    // Handler 2: Withdraw Cash
    on<WithdrawCashEvent>((event, emit) async {
      emit(AtmProcessingState('Counting physical bills and validating vault...'));
      await Future.delayed(const Duration(milliseconds: 1200));

      if (event.amount > _accountBalance) {
        emit(AtmErrorState('Declined: Insufficient funds. Available: \$$_accountBalance'));
        return;
      }

      _accountBalance -= event.amount;
      emit(AtmDispensedSuccessState(
        amountDispensed: event.amount,
        remainingBalance: _accountBalance,
      ));
    });

    // Handler 3: Eject Card
    on<EjectCardEvent>((event, emit) {
      emit(AtmIdleState());
    });
  }
}

// ============================================================================
// 5. CUSTOM BLOC BUILDER WIDGET
// ============================================================================
class CustomBlocBuilder<B extends CustomBloc<dynamic, S>, S> extends StatefulWidget {
  final B bloc;
  final Widget Function(BuildContext context, S state) builder;

  const CustomBlocBuilder({super.key, required this.bloc, required this.builder});

  @override
  State<CustomBlocBuilder<B, S>> createState() => _CustomBlocBuilderState<B, S>();
}

class _CustomBlocBuilderState<B extends CustomBloc<dynamic, S>, S>
    extends State<CustomBlocBuilder<B, S>> {
  late S _currentState;
  StreamSubscription<S>? _subscription;

  @override
  void initState() {
    super.initState();
    _currentState = widget.bloc.state;
    _subscription = widget.bloc.stream.listen((newState) {
      if (mounted) setState(() => _currentState = newState);
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return widget.builder(context, _currentState);
  }
}

// ============================================================================
// 6. ROOT APPLICATION
// ============================================================================
class BlocMasteryApp extends StatefulWidget {
  const BlocMasteryApp({super.key});

  @override
  State<BlocMasteryApp> createState() => _BlocMasteryAppState();
}

class _BlocMasteryAppState extends State<BlocMasteryApp> {
  final AtmBloc _atmBloc = AtmBloc();

  @override
  void dispose() {
    _atmBloc.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLoC Architecture',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorSchemeSeed: Colors.indigo,
        useMaterial3: true,
      ),
      home: AtmTerminalScreen(atmBloc: _atmBloc),
    );
  }
}

// ============================================================================
// 7. ATM TERMINAL SCREEN: Pure UI mapping States to visual layouts
// ============================================================================
class AtmTerminalScreen extends StatelessWidget {
  final AtmBloc atmBloc;

  const AtmTerminalScreen({super.key, required this.atmBloc});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Bank of Flutter ATM Terminal'),
        centerTitle: true,
      ),
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(24.0),
          child: CustomBlocBuilder<AtmBloc, AtmState>(
            bloc: atmBloc,
            builder: (context, state) {
              // PATTERN MATCHING ON EMITTED STATES!
              if (state is AtmIdleState) {
                return _buildIdleView();
              } else if (state is AtmProcessingState) {
                return _buildProcessingView(state.message);
              } else if (state is AtmCardInsertedState) {
                return _buildCardInsertedView(state);
              } else if (state is AtmDispensedSuccessState) {
                return _buildSuccessView(state);
              } else if (state is AtmErrorState) {
                return _buildErrorView(state.errorMessage);
              }
              return const SizedBox.shrink();
            },
          ),
        ),
      ),
    );
  }

  // State 1: Idle
  Widget _buildIdleView() {
    return Card(
      elevation: 4,
      child: Padding(
        padding: const EdgeInsets.all(32.0),
        child: Column(
          children: [
            const Icon(Icons.credit_card, size: 72, color: Colors.indigo),
            const SizedBox(height: 16),
            const Text('Welcome to Global ATM', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text('Please insert your chip debit card to begin.', style: TextStyle(color: Colors.grey)),
            const SizedBox(height: 24),
            FilledButton.icon(
              onPressed: () {
                // DISPATCH EVENT!
                atmBloc.add(InsertCardEvent('4532789012349081'));
              },
              icon: const Icon(Icons.input),
              label: const Text('Insert Debit Card (4532...9081)'),
            ),
          ],
        ),
      ),
    );
  }

  // State 2: Processing Spinner
  Widget _buildProcessingView(String message) {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(40.0),
        child: Column(
          children: [
            const CircularProgressIndicator(),
            const SizedBox(height: 24),
            Text(message, textAlign: TextAlign.center, style: const TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text('Please do not remove your card...', style: TextStyle(color: Colors.grey, fontSize: 12)),
          ],
        ),
      ),
    );
  }

  // State 3: Card Inserted / Ready for Transaction
  Widget _buildCardInsertedView(AtmCardInsertedState state) {
    return Card(
      elevation: 4,
      child: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          children: [
            ListTile(
              leading: const Icon(Icons.account_balance_wallet, color: Colors.green, size: 36),
              title: Text('Card: ${state.maskedCard}'),
              subtitle: Text('Balance: \$${state.availableBalance.toStringAsFixed(2)}'),
            ),
            const Divider(height: 32),
            const Text('Select Quick Cash Amount:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 16),
            Wrap(
              spacing: 12,
              runSpacing: 12,
              children: [
                _cashBtn(50),
                _cashBtn(100),
                _cashBtn(200),
                _cashBtn(1000), // Will trigger insufficient funds!
              ],
            ),
            const SizedBox(height: 24),
            OutlinedButton.icon(
              onPressed: () => atmBloc.add(EjectCardEvent()),
              icon: const Icon(Icons.eject),
              label: const Text('Eject Card & Cancel'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _cashBtn(double amount) {
    return FilledButton.tonal(
      onPressed: () => atmBloc.add(WithdrawCashEvent(amount)),
      child: Text('\$${amount.toInt()}'),
    );
  }

  // State 4: Success
  Widget _buildSuccessView(AtmDispensedSuccessState state) {
    return Card(
      color: Colors.green.shade50,
      elevation: 3,
      child: Padding(
        padding: const EdgeInsets.all(32.0),
        child: Column(
          children: [
            const Icon(Icons.check_circle, size: 64, color: Colors.green),
            const SizedBox(height: 16),
            Text('Dispensed \$${state.amountDispensed.toStringAsFixed(2)}', style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Text('Remaining Account Balance: \$${state.remainingBalance.toStringAsFixed(2)}'),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => atmBloc.add(EjectCardEvent()),
              child: const Text('Take Cash & Finish'),
            ),
          ],
        ),
      ),
    );
  }

  // State 5: Error
  Widget _buildErrorView(String message) {
    return Card(
      color: Colors.red.shade50,
      elevation: 3,
      child: Padding(
        padding: const EdgeInsets.all(32.0),
        child: Column(
          children: [
            const Icon(Icons.error_outline, size: 64, color: Colors.red),
            const SizedBox(height: 16),
            const Text('Transaction Error', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.red)),
            const SizedBox(height: 8),
            Text(message, textAlign: TextAlign.center),
            const SizedBox(height: 24),
            FilledButton(
              onPressed: () => atmBloc.add(EjectCardEvent()),
              child: const Text('Return Card to Idle'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 🔍 Line-by-Line Example Explanation

- **Lines 10–23 (`AtmEvent`)**: Defines all possible user inputs. Notice that events are immutable classes that describe *what happened*, not *what to do*.
- **Lines 28–53 (`AtmState`)**: Defines all mutually exclusive UI states: `AtmIdleState`, `AtmProcessingState`, `AtmCardInsertedState`, `AtmDispensedSuccessState`, and `AtmErrorState`.
- **Lines 90–129 (`AtmBloc`)**:
  - `on<InsertCardEvent>`: Emits `AtmProcessingState`, waits 1 second, and emits `AtmCardInsertedState`.
  - `on<WithdrawCashEvent>`: Emits processing, checks if balance is sufficient, and emits either `AtmDispensedSuccessState` or `AtmErrorState`.
  - Notice that all business rules (checking balance, deducting funds) are 100% inside the BLoC!
- **Lines 195–215 (`AtmTerminalScreen.build`)**:
  - The UI uses `CustomBlocBuilder`.
  - Notice the pattern:
    ```dart
    if (state is AtmIdleState) return _buildIdleView();
    if (state is AtmCardInsertedState) return _buildCardInsertedView(state);
    ...
    ```
    The UI is a **pure finite state machine**. It doesn't track any booleans or variables; it simply renders whichever widget matches the emitted state!

---

## 📱 ASCII Phone Screen Preview

```
┌──────────────────────────────────────┐
│       Bank of Flutter ATM Terminal   │
├──────────────────────────────────────┤
│                                      │
│  ┌────────────────────────────────┐  │
│  │             [ 💳 ]              │  │
│  │     Welcome to Global ATM      │  │  <-- State: AtmIdleState
│  │ Please insert your debit card  │  │
│  │                                │  │
│  │ [ Insert Debit Card (..9081) ] │  │  <-- Tapping dispatches
│  └────────────────────────────────┘  │      InsertCardEvent
│                                      │
└──────────────────────────────────────┘
                   │ User inserts card -> Emits ProcessingState -> CardInsertedState
                   ▼
┌──────────────────────────────────────┐
│  ┌────────────────────────────────┐  │
│  │ [💼] Card: •••• 9081           │  │  <-- State: AtmCardInsertedState
│  │      Balance: $850.00          │  │
│  │ ────────────────────────────── │  │
│  │ Select Quick Cash Amount:      │  │
│  │ [$50]   [$100]  [$200]  [$1000]│  │
│  │                                │  │
│  │    [ ⏏ Eject Card & Cancel ]   │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
                   │ User taps [$1000] (Insufficient funds!)
                   ▼
┌──────────────────────────────────────┐
│  ┌────────────────────────────────┐  │
│  │             [ ⚠ ]              │  │
│  │       Transaction Error        │  │  <-- State: AtmErrorState
│  │ Declined: Insufficient funds.  │  │
│  │                                │  │
│  │    [ Return Card to Idle ]     │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

---

## 🌳 Widget Tree

```
BlocMasteryApp
 └── AtmTerminalScreen (StatelessWidget)
      └── Scaffold
           └── Center
                └── SingleChildScrollView
                     └── CustomBlocBuilder<AtmBloc, AtmState>
                          └── (Renders either Idle, Processing, Inserted, Success, or Error Card)
```

---

## ⚙ Flutter Execution Flow

```
User taps "$100" quick cash button
                    │
                    ▼
1. Event dispatched: `atmBloc.add(WithdrawCashEvent(100))`
                    │
                    ▼
2. BLoC handler executes: `emit(AtmProcessingState(...))`
                    │
                    ▼
3. Stream emits: Screen repaints to show CircularProgressIndicator
                    │
                    ▼
4. BLoC checks balance: $850 >= $100 -> Success!
   Deducts $100 -> New balance: $750.00
                    │
                    ▼
5. BLoC executes: `emit(AtmDispensedSuccessState(100, 750))`
                    │
                    ▼
6. Stream emits: Screen repaints with green "Dispensed $100.00" card!
```

---

## ⚠ Common Beginner Mistakes

### 1. Putting Navigation or Dialogs Inside `BlocBuilder`

❌ **Fatal Error**:
```dart
BlocBuilder<AuthBloc, AuthState>(
  builder: (context, state) {
    if (state is AuthSuccess) {
      // CRASH! Never trigger side effects inside a build method!
      Navigator.pushNamed(context, '/home');
    }
    return Container();
  },
)
```

✔ **Correct**: Use `BlocListener` for side effects (navigation, SnackBars, dialogs):
```dart
BlocListener<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthSuccess) {
      Navigator.pushNamed(context, '/home'); // Safe!
    }
  },
  child: ...,
)
```

---

### 2. Modifying State Variables Inside BLoC Methods

❌ **Wrong**: Declaring public mutable variables in a BLoC and changing them directly.
✔ **Correct**: The BLoC must emit new immutable state instances using `emit()`.

---

### 3. Adding Events Inside the `build()` Method

❌ **Infinite Loop Crash**: Dispatching `bloc.add(FetchData())` directly in `build()`. Every rebuild dispatches an event, which emits a state, which rebuilds the widget infinitely!

---

## 🚀 Professional Usage: The `BlocObserver` (Global Logging)

In enterprise banking and healthcare Flutter apps, developers install a global `BlocObserver` to record every event and state transition across the entire system:

```dart
class AppBlocObserver extends BlocObserver {
  @override
  void onEvent(Bloc bloc, Object? event) {
    super.onEvent(bloc, event);
    print('📥 [EVENT] ${bloc.runtimeType} -> $event');
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print('🔄 [TRANSITION] ${bloc.runtimeType}: ${transition.currentState} -> ${transition.nextState}');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('❌ [ERROR] ${bloc.runtimeType} -> $error');
    super.onError(bloc, error, stackTrace);
  }
}

// In main.dart:
void main() {
  Bloc.observer = AppBlocObserver(); // 100% full application observability!
  runApp(const MyApp());
}
```

---

## 💡 Best Practices Checklist

- [ ] **Events in, States out**: Keep the contract clean and unidirectional.
- [ ] **Use `BlocBuilder` for UI rendering** and **`BlocListener` for navigation & SnackBars**.
- [ ] **Use `BlocConsumer`** when you need both in the same widget.
- [ ] **State must be immutable**: Subclass states and use `Equatable` or `copyWith()`.
- [ ] **Install a `BlocObserver`** for global debugging and crash reporting.

---

## 🧪 Try It Yourself

1. Tap **"Insert Debit Card"**: Observe the 1-second processing screen before the card dashboard appears.
2. Tap **"\$200"**: Watch the ATM deduct \$200 and transition to the green success screen with \$650 remaining balance.
3. Insert the card again and tap **"\$1000"**: Observe the red "Declined: Insufficient funds" error screen trigger automatically!

---

## 🎯 Challenge

Build a **Stopwatch BLoC**:
- Events: `StartTimer`, `PauseTimer`, `ResetTimer`, `TickTimer(duration)`.
- States: `TimerInitial`, `TimerRunning(duration)`, `TimerPaused(duration)`.
- Inside `on<StartTimer>`, subscribe to a `Stream.periodic` that emits `TickTimer` events every second!
- Render the stopwatch UI with `BlocBuilder`.

---

## ❓ Practice Questions

### Q1. What is the fundamental philosophy of the BLoC pattern in Flutter?
**Answer**: BLoC (Business Logic Component) is an event-driven architectural pattern where all data flows unidirectionally through Streams: UI widgets dispatch **Events** into the BLoC, the BLoC processes business rules and communicates with repositories, and the BLoC emits immutable **States** back to the UI to trigger rebuilds.

---

### Q2. What is the difference between `BlocBuilder` and `BlocListener`?
**Answer**: `BlocBuilder` is responsible for building visual widgets in response to new states (it re-executes its `builder` callback). `BlocListener` is responsible for executing one-time **side effects** (like navigating to a route, showing a `SnackBar`, or opening a dialog) and does *not* rebuild widgets.

---

### Q3. When should you use `BlocConsumer`?
**Answer**: `BlocConsumer` combines both `BlocBuilder` and `BlocListener` into a single widget. You should use it when a screen needs to rebuild its UI *and* execute a side effect (such as displaying an error toast) in response to the same state change.

---

### Q4. Why should you NEVER navigate (`Navigator.push`) inside a `BlocBuilder`?
**Answer**: `BlocBuilder`'s callback is a pure build function that executes during Flutter's layout pipeline. Triggering navigation or showing dialogs during a build phase violates framework lifecycle rules and throws the assertion error: `setState() or markNeedsBuild() called during build`.

---

### Q5. What is a `BlocObserver`?
**Answer**: `BlocObserver` is a centralized monitoring class in the `bloc` package that intercepts all events, transitions, and errors across every BLoC in the entire application, enabling global debugging, analytics logging, and crash monitoring.

---

### Q6. How does modern BLoC (8.0+) register event handlers?
**Answer**: Using the `on<EventType>((event, emit) async { ... })` method inside the BLoC constructor, replacing older `mapEventToState` generator syntax.

---

### Q7. What is an event transformer in BLoC?
**Answer**: An event transformer allows developers to alter how events are processed (e.g. using `droppable()` to drop duplicate taps while a task is running, or `debounceTime()` to pause search input queries).

---

### Q8. What package is commonly used with BLoC to ensure value equality between states?
**Answer**: The `equatable` package. By extending `Equatable`, BLoC can determine if two distinct state instances have identical fields, suppressing redundant widget rebuilds.

---

### Q9. Can a BLoC emit multiple states from a single event?
**Answer**: Yes! An event handler can call `emit()` multiple times (e.g., emitting `LoadingState()` immediately, and then emitting `SuccessState()` after an `await` network call finishes).

---

### Q10. What is the difference between a BLoC and a Cubit?
**Answer**: A BLoC is strictly event-driven (requires creating Event classes and calling `bloc.add(Event)`). A `Cubit` is a simplified subset of BLoC that eliminates Event classes, allowing widgets to trigger state transitions by calling public methods directly (`cubit.increment()`).

---

## 📝 Summary

- **Event-Driven Purity**: [Events] enter the BLoC $\rightarrow$ [States] exit the BLoC.
- **The Core Widgets**: `BlocProvider` (injects), `BlocBuilder` (paints UI), `BlocListener` (side effects), and `BlocConsumer` (both).
- **Zero Business Logic in UI**: Widgets only dispatch events and render states.
- **Global Observability**: Use `BlocObserver` to log and trace every transition in the app.
- **Enterprise Ready**: The standard architecture for mission-critical banking, enterprise, and complex Flutter applications.
