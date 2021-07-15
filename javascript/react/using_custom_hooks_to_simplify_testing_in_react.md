# Using custom hooks to simplify testing in React

- Author: Sam Bland
- Date:
- Status: WIP
- Software: React/Javascript/Jest
- Theme: Testing


Isolating your tests in react can often be difficult and involve complex mocking and hacks.
By taking a functional approach to our components to ensure each part can be tested in isolation we can simplify our testing.
Take the following example react component

``` js
const AsyncComponent = ({
  callback,
}) => {
  const [flag, setFlag] = useState(false);

  useEffect(() => {
    if (flag) {
      setFlag(false);
      asyncAction().then((data) => {
        if (data) {
          callback();
        }
      });
    }
  }, [flag, callback]);

  return (
    <div>
      <button
        type="button"
        className="button-one"
        onClick={() => setFlag(true)}
      >
        Click Me
      </button>
    </div>
  );
};

```

We want to test that when the button is clicked it updates the state. Once the state has updated it will call the async action inside the use effect. When that async action returns it calls the callback prop with any returned data.


That is a lot of asynchronous activity to follow and test. We could mock the async action but that comes with its own issues depending on how you export your async actions if you want to assign a jest mock to the function to track its use.


The first step in my refactoring involved passing the asyncAction function directly in as a prop. That way we can easily mock it by simply passing any other function in within the test file. We set the imported function as the default so that when using the component we can ignore this logic.

``` js

const AsyncComponent = ({
  // Props
  callback,
  // Functions
  asyncAction = asyncActionApi
}) => {
  const [flag, setFlag] = useState(0);

  useEffect(() => {
    if (flag) {
      setFlag(false);
      asyncAction().then((data) => {
        if (data) {
          callback();
        }
      });
    }
  }, [flag, callback, asyncAction]);

  return (
    <div>
      <button
        type="button"
        className="button-one"
        onClick={() => setFlag(true)}
      >
        Click Me
      </button>
    </div>
  );
};
```
This has now removed 1 of the async events. To test this component we simply pass in our mocked function as below.

``` js

describe('Component', () => {
  const demoData = { foo: 'foo' };
  const mockApiAsyncFn = jest.fn().mockImplementation(async () => demoData);
  const mockCallback = jest.fn();
  it('Calls async function then callback', () => {
    const component = mount(
      <AsyncComponent
        callback={mockCallback}
        asyncAction={mockApiAsyncFn}
      />,
    );

    expect(mockApiAsyncFn).toHaveBeenCalledTimes(0);
    expect(mockCallback).toHaveBeenCalledTimes(0);

    const btn = component.find('button');
    btn.simulate('click');
    expect(mockApiAsyncFn).toHaveBeenCalledTimes(1);
    expect(mockCallback).toHaveBeenCalledTimes(1);
  });
});



```

To remove the async elements of the component completely leaving our component to manage how it renders and links up ui elements we can implement a custom hook.

The following custom hook will perform all the logic from the component leaving the component to render the output.

``` js

export const useAsyncThingFn = (callback) => {
  const [data, setData] = useState(0);
  const [flag, setFlag] = useState(false);

  const doThing = () => {
    setFlag(true);
  };

  const doAsyncThing = async () => {
    await timeout(100);
    setData(1);
    callback();
  };

  useEffect(() => {
    if (flag) {
      setFlag(false);
      setData(0);
      doAsyncThing();
    }
  }, [flag]);

  return { data, doThing };
};

```

To implement this in our component simply use like a standard react hook...
``` js


const AsyncComponent = ({
  // Props
  callback,
  // Functions
  useAsyncThing = useAsyncThingFn,
}) => {
  const { data, doThing } = useAsyncThing(callback);

  return (
    <div>
      <button
        type="button"
        className="button-one"
        onClick={doThing}
      >
        Click Me
      </button>
    </div>
  );
};

```
Our testing can then be split between the UI and async elements of the component.

Test the component as below:

``` js
...

describe('Component', () => {
  const demoData = { foo: 'foo' };
  const mockCallback = jest.fn();
  const mockUseAsyncThing = (callback) => ({ state: demoData, doThing: () => callback() });
  it('Calls async function then callback', () => {
    const component = mount(
      <AsyncComponent
        callback={mockCallback}
        useAsyncThing={mockUseAsyncThing}
      />,
    );

    expect(mockCallback).toHaveBeenCalledTimes(0);
    const btn = component.find('button');
    btn.simulate('click');
    expect(mockCallback).toHaveBeenCalledTimes(1);
  });
});

```

To test the hook we need to use the react testing-library/react-hooks library as below.

``` js
...
import { renderHook, act } from '@testing-library/react-hooks';

describe('Hook', () => {
  test('should update', (done) => {
    const fn = jest.fn();
    const { result, waitForNextUpdate } = renderHook(() => useAsyncThingFn(fn));
    expect(result.current.data).toEqual(0);
    expect(typeof result.current.doThing).toBe('function');
    act(() => {
      result.current.doThing();
    });
    expect(result.current.data).toEqual(0);
    expect(fn).toHaveBeenCalledTimes(0);
    waitForNextUpdate()
      // We wait for next update to ensure the use effect has ran after the flag was changed.
      .then(() => {
        expect(result.current.data).toEqual(1);
        expect(fn).toHaveBeenCalledTimes(1);
        done();
      });
  }, 3000);
});


```

The bonus to the second method is we can then reuse that hook elsewhere!