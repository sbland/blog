Prior to react hooks testing your react components relied on testing render responses to user input. This is far from the ideal of isolated unit tests and could often have untested side effects.

Take the following React component:


``` js
const AsyncComponent = ({
}) => {
  const [flag, setFlag] = useState(false);
  const [response, setResponse] = useState(null);
  const [loading, setLoading] => useState(false);

  useEffect(() => {
    if (flag) {
      setFlag(false);
      setLoading(true);
      asyncAction().then((data) => {
        if (data) {
          setResponse(data);
          setLoading(false);
        }
      });
    }
  }, [flag, loading, response]);

  return (
    <div>
      (loading && (
        <p className="loadingWrap">
            Is loading
        </p>
      ))
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

In the above we can test the following flow:
`[Button Click] => [Shows Loading] => [Async Action] => []