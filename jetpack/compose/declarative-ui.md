# Declarative UI

### Imperative vs Declarative

#### Imperative

하나의 뷰 작성
이 뷰의 여러 API를 이용해서 뷰의 상태를 변화시킴으로써 UI 변경
Well.. fine..?
Animation? Transition? WTF

```kotlin
val button = createNewButton()
val container = binding.container
container.removeAllViews()
container.setBackgroundColor(Color.RED)
container.addView(button)
```

#### Declarative

원하는 상태의 청사진을 작성
상태를 명시하면 뷰가 알아서 UI 변경

```kotlin
return State(
    backgroundColor = Color.RED,
    children = [
        createNewButton()
    ]
)
```

##### React

```react
import randomWords from "random-words";

class RandomWords extends React.Component {
  constructor(props) {
    super(props);
    this.state = {word: randomWords()};
  }
  
  randomWords() {
    
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>The word is [{this.state.word}]</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <RandomWords />,
  document.getElementById('root')
);
```

##### Flutter

```dart
class RandomWordsState extends State<RandomWords> {
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();
    return Text(wordPair.asPascalCase);
  }
}

class RandomWords extends StatefulWidget {
  @override
  RandomWordsState createState() => RandomWordsState();
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(            
      title: 'Welcome to Flutter',
      home: Scaffold(
        body: Center(
          child: RandomWords(),
        ),
      ),
    );
  }
}
```