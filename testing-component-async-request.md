# Unit-Testing asynchronous data fetching on a React component

The following article is a direct result of the code review process that we run internally in our team. I was implementing a new feature, in which I decided to fetch data directly from the component and not to go over the redux actions and store. I guess we can have a really long discussion just about whether we should be fetching data directly from the component or over redux. But in my case, I decided to keep it simple and just fetch the data I need where I need it.

On the pull request, I got the feedback that if I used an Action it would be easier to test. Hmm... This read to me like a challenge, so I decided to test this scenario as is so that I'm able to write more of these tests in the future.

Regardless to what we might be using for state management (or not), the following is quite a typical scenario in a React component: We mount a component with a property that serves as an id. Then in `componentDidMount` we use that prop in order to fetch some data and set the component's state accordingly.

```
import React from 'react';
import fetch from 'isomorphic-fetch';

class Weather extends React.Component {
  state = {
    weather: {
      summary: null
    },
  };

  componentDidMount() {
    fetch(`https://weather.example.com/api/weather?q=${this.props.location}`)
      .then(res => res.json())
      .then((weather) => {
        this.setState({
          weather,
        });
      });
  }

  render = () => (
    <span>
      The weather in {this.props.location} is: {this.state.weather}
    </span>
  );
}
```

The component above is actually quite tricky to test. This is because `componentDidMount` does some asynchronous work, what means that the execution of `componentDidMount` will terminate before the network response reaches the component and we don't have any guarantee when or even if the `setState` function will get invoked.

## The test case

First, let's define what are we testing: We have a component that when mounted, will shoot an HTTP request to our imaginary weather API and then set the weather conditions received into its state.

Since we're not supposed to use a real network connection in a unit-test scenario we will mock the request so that we'll have absolute control on the response. This will allow us to assert that the component behaves the way we expect by looking into its state and comparing it to the mocked http response. We could also test for what the component actually renders. I feel that is unnecessary because it's too trivial. If we assert on the state that should be enough. Besides, if we ever decide to change the message just a tiny bit our test will fail. The way the state is handled, however, is not very likely to change so much and therefore I find it a better candidate for the assertion in our test.

## The solution we need

In our unit test, if we check the state too early, the data might not be there yet and our test will fail. On the other hand, if we just wait for a fixed amount for the response to arrive at the component, we are wasting precious time and making our unit tests slow. Both scenarios are unacceptable since unit-tests must be reliable _and_ fast.

There are some thumb rules that hold for all unit tests:

1. Test results must be deterministic and not affected by the environment. This means that if a unit test is failing or passing, repeating the test cannot change this, not even in a million years. If a unit-test is flaky then it's either doing something it shouldn't like a network request for example or suffering from another kind of complex racing-conditions. Either way, a unit test that fails _sometimes_ is something we can't accept.

2. A unit-test must be quick and robust. Using a fixed time to wait for something is not going to provide for these conditions. Either we will wait very long for the response, which would make our test slow, or we don't wait enough which will make our test unreliable.

3. We don't mock the unit under test. I read people suggesting to mock the internal functions of the component under test and then assert that these functions were called. But the way I see it, if you change anything in the component that you are testing, the test isn't valid anymore since you are no longer testing your unit but your _modification_ of that unit. The idea here is to test that a unit works as we expect over a clear API and not to mess with its internals.

## The tools we'll be using to achieve this

In this example, I'll be using two additional tools relevant to the task at hand. There are probably other npm modules out there that do the same, perhaps even better. More than the tools themselves it is important that you develop the understanding of what are we trying to achieve by using these tools.

### nock - HTTP server mocking and expectations library for Node.js

[nock](https://www.npmjs.com/package/nock) is a solid library under active maintenance for manipulating and asserting on HTTP responses and requests. In our case, nock serves a double purpose:

#### The component needs to think that the network connection is working.
Remember we're in a unit-test scenario that means that the execution of the tests is not allowed to generate any actual network activity. That's why they're called unit tests because we're testing just our unit and nothing else but our unit. We are not testing that our internet connection works properly. That's not in our concern for now. nock helps us to hijack the request to the weather API and prevent it from going out to the internet.

#### We want to control the response 
When writing a test we must ensure a deterministic and repeatable behavior. We cannot allow any external conditions like the size of CPU to break our tests. We definitely can't allow a change in the weather to them! By mocking the HTTP request, we can inject the response that will be received by our component under test. Yes, we can make London sunny all year long!

### async-wait-until

The second tool we'll be using is this small library called [async-wait-until](https://github.com/devlato/waitUntil). This modern library is based on Promises and has TypeScript support, two things that we love! In the simplest use-case, we provide `waitUntil` with a predicate. waitUntil will keep executing the predicate until it's true. Once the predicate function returns true the promise will resolve.

For example, the following simple lines will keep looking at the watch until it's 4 pm and make us coffee.

```
waitFor(
  () => new Date().getHours() === 16
).then(makeCoffee)
```

Quite useful isn't it?


## The test

Getting back to our weather component and using nock and async-wait-until, here's what our test looks like:

```jsx
import waitUntil from 'async-wait-until';
import { shallow } from 'enzyme';
import nock from 'nock';
import React from 'react';
import Weather from '../Weather';

describe('<Weather />', () => {
  beforeAll(() => {
    // Prepare nock to respond to a request
    // to the weather API.
    // In this case our test will always think that london
    // is sunny.
    nock('https://weather.example.com/api')
      .get('/weather?q=london')
      .reply(200, {
           summary: 'sunny',
      });
  });

  it('Component fetching weather from API', async (done) => {
    const root = shallow(<Weather location="london" />);

    let componentsWeather = {};
    
    // We wait until the state has a weather summary, but we
    // don't care yet about the content.
    await waitUntil(() => root.state('weather').summary !== null);

    // It is better to have the expectation here and not inside
    // the waitUntil condition.
    expect(componentsWeather.summary).toEqual('sunny');

    done();
  });
});

```

As you can see in the test above we have absolute control about what our endpoint returns. It is important that your unit tests will use a domain that doesn't exist in reality. This way you can be certain that the unit tests aren't getting any data from the internet. You can achieve this via configuration of your application.