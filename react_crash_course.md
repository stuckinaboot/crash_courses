# React Crash Course

Note: _My team found this to be a great React primer and so I decided to open source it. If you have suggestions, feel free to submit a PR :)_

React is a framework for building web frontends. While there can be a learning curve to react, this crash course is designed to alleviate that learning curve. My first recommendation is to download the [VS Code](https://code.visualstudio.com/) as it is **extremely** helpful in creating React components and debugging issues related to React components. You are doing yourself a disservice if you do not write React code using [VS Code](https://code.visualstudio.com/). Also, in all of the examples, there are comments for clarity but the react commenting style is a bit different so remove these if you copy-paste.

## Key Concepts

The key concepts in React are components, properties (props), and state.

**Component**: Class or function that optionally accepts inputs (properties) and returns a React element that describes how a section of the UI should appear.

**Properties**: Inputs to a react component that influence the component's behavior and possibly how the component will appear on the UI. For example, the title of a component can be a property.

**State**: An object that represents parts of a component that can change. For example, if a component has a button, then you could have a counter (which is state) that increments each time the button is pressed.

Here is a simple example component, implemented using a function. Note the `export default` which is used to export the component from the file (to be the component available to other files):
```
// This is a react component
export default function InfoCard() {

  // This is a react element
  return (
    <Card>
      <CardContent>
        <Typography>
          Some words
        </Typography>
        <Typography>
          Some other words
        </Typography>
      </CardContent>
    </Card>
  );
}
```

## Intro

### Using a react component

We could use our InfoCard component in the example above in some other react component. For example, we can have a component BigCard, that contains our InfoCard:

```
export default function BigCard() {
  return (
    <div>
      <InfoCard /> // which is equivalent to <InfoCard><InfoCard />
      <Typography>Here is some random text below our info card component</Typography>
    </div>
  );
}
```


### Rendering values in components

Typically, you'll want to include some dynamic values in the component. To do this, we place any typescript that we want to evaluate within { ... }. For example, if you have a variable `foo` and you want to represent the value of `foo` in a react element, you would write `{foo}`. Here we pass properties as input to the component so that we can render them as part of the component.

```
export default function InfoCard(props: { name: string; description: string }) {
  return (
    <Card>
      <CardContent>
        <Typography>
          // This evaluates props.name so that the value of props.name will show
          {props.name}
        </Typography>
        <Typography>
          // This evaluates props.description so that the value of props.description will show
          {props.description}
        </Typography>
      </CardContent>
    </Card>
  );
}
```

### Rendering arrays in components

In order to render an array in a react component, you need to map each element in that array to react component.
```
export default function InfoCard(props: { names: string[]; }) {
  // props.names.map maps each name (string) in props.names to a react element
  return (
    <Card>
      <CardContent>
        <div>
          {props.names.map(name => (
              <Typography>{name}</Typography>
            )
          )}
        </div>
      </CardContent>
    </Card>
  );
}
```

## Event handlers in React components

React components support certain events, with the most important one's being `useEffect` and `useState`. 

### useEffect

The `useEffect` handler (first argument) is called automatically when the react component loads or when the second argument to `useEffect` changes in value. If you only want `useEffect` to be called once (e.g. when the component is loaded), pass `[]` as the second argument. If you provide no second argument, the component may repeatedly call `useEffect` until it is removed, so remember to provide an argument.

#### useEffect will only be called once

```
export default function MyReactComponent() {
  useEffect(() => {
    console.log("Woohoo!");
  }, []);

  return <div>Some words</div>;
}
```

#### useEffect will be called whenever the value of props.foo is changed

If the variable that is passed into MyReactComponent changes in value, then the handler to useEffect (the first argument to useEffect) will be called again. This is useful as this allows the parent component (e.g. a component that contains <MyReactComponent />) to update the state of its children when one of its own variables changes.

```
export default function MyReactComponent(props: { foo : boolean }) {
  useEffect(() => {
    console.log("Woohoo!");
  }, [props.foo]);

  return <div>Some words</div>;
}
```

#### useEffect and async functions

Often you'll want to perform some async function when the component is first loaded, such as requesting data from the SixFeetAway server. To do this, you create an async function and call that function from the useEffect handler.

```
export default function MyReactComponent(props: { foo : boolean }) {

  useEffect(() => {
    async function myAsyncFn() {}
    myAsyncFn();
  }, []);

  return <div>Some words</div>;
}
```

### useState

State is one of the useful parts of react and its what allows react components to update dynamically without you having to do anything to change the UI. In order to create state in react, you use the following syntax:
`const [myState, setMyState] = useState("" /* some default value*/);`

Then, when you want to change the value of `myState`, you call `setMyState("some new value");`. The reasoning for this is that calling `setMyState` tells react that the UI may need to be refreshed. **`myState = "some value";` will not correctly update the UI so don't do that**.

Here is a simple example of how state can be used

#### State on its own

```
export default function MyContentsComponent() {
  const [contents, setContents] = useState("");
  
  if (contents === "") {
    setContents("contents will now equal this string");
  }

  return <div>{contents}</div>;
}
```

#### State with useEffect

You will typically want to alter state when some action occurs, such as a component being loaded. Here is an example of how you would set state after performing a request and then using that state in the react component.

```
export default function MyContentsComponent() {
  const [contents, setContents] = useState("");

  useEffect(
    () => {
      async function getFileContents() {
        const res = await fetch("/some_endpoint");
        const resTxt = await res.text();
        setContents(resTxt);
      }

      getFileContents();
    }, []
  );

  return <div>{contents}</div>;
}
```

### Parent component - child component interaction

#### Passing values into children

`MyParentComponent.tsx`:
```
export default function MyParentComponent(props: { accountId: string }) {
  // Note how we evaluate the variable props.accountId by placing it in { ... } in order to pass it into MyChildComponent
  return (
    <div>
      <MyChildComponent name="John" accountId={props.accountId} />
    </div>
  );
}
```

`MyChildComponent.tsx`:
```
export default function MyChildComponent(props: { name: string; accountId: string }) {
  // Display the input name and accountId with a space between them
  return (
    <div>
      {props.name} {props.accountId}
    </div>
  );
}
```

#### Passing callbacks into children

Sometimes you will want the parent component to be notified when certain events happen in the child component. An example of this would be notifying the parent component when a button was pressed in the child component. Here is an example of how this could be accomplished by passing a callback function from the parent to the child through properties.

`MyParentComponent.tsx`:
```
export default function MyParentComponent(props: { accountId: string }) {
  function printWoohoo() {
    console.log("Woohoo!");
  }

  // Note how the function printWoohoo needs to be placed in { ... } in order to be passed into MyChildComponent
  return (
    <div>
      <MyChildComponent onButtonPress={printWoohoo} />
    </div>
  );
}
```

`MyChildComponent.tsx`:
```
export default function MyChildComponent(props: { onButtonPress: (event: any) => any }) {
  // When the button is pressed, the input function props.onButtonPress, which is in the parent component, will be called
  return (
    <button onClick={props.onButtonPress}>Cool button</button>
  );
}
```
