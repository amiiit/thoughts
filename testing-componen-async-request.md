# Unit-Testing asynchronous data fething on a React component

The following is quite a typical scenario in React component that get data over network request. Say we have the following component:

```
import React from 'react';
import fetch from 'isomorphic-fetch';

class Weather extends React.Component {
  state = {
    weather: '',
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

Can you see why this is hard to test? This is because componentDidMount works asynchronously, what means that its executing will end before the network request terminates and we don't have any guarantee when or even if the `setState` function will get invoked.

## The test case

What we would like to test here is quite simple. When mounted, the component needs to shoot an HTTP request to the weather API and when provided a certain response it should set it into its state.

Since we're not supposed to use a real network connection in a unit-test scenario we will mock the HTTP request so we will have absolute control on the response. This will allow us to assert that the component will display the right thing according to the weather returned from our unit test.

## The solution we need

There are some conditions that need to be fulfilled in order to this to be a real unit test:

1. It must be quick and robust. Using `setTimeout` is not going to provide for both these conditions. Either we will wait too long for the response, which would make our test slow, or we don't wait enough which will make our test fail sometimes.
2. We don't mock the unit under test. I read people suggesting to mock the internal functions of the component under test and then assert that these functions were called. But the way I see it, if you change anything in the unit that you are testing, this is no longer a valid unit test. The idea here is to test that the component works as we expect, and not to test the implementation details.

## The tools we'll be using to achieve this

### nock - HTTP server mocking and expectations library for Node.js

[nock](https://www.npmjs.com/package/nock) is a solid library under active maintenance for manipulating and HTTP responses and asserting on requests. In our case, this serves a double purpose:

#### The component needs to think that the network connection is working.
Remember we're in a unit-test scenario that means that the unit tests shuld pass even if we have no network connection. That's why they're called unit tests, because we're testing our unit, which in this case is the React component. We are not testing that our internet connection works properly. That's not in our concern for now. nock helps us to hijack the request to the weather API and prevent it from going

#### We want to control the response 
When writing a test we must ensure a deterministic behavior. Even though some places do have mostly bad weather, we can't allow one sunny day to break all our unit tests. With nock, or any other HTTP mocking library, we can set the response that will be received by our component under test. Yes, we can even make London sunny all year long!

## The test

Here's what the test would look like.

```jsx
import waitUntil from 'async-wait-until';
import { shallow } from 'enzyme';
import nock from 'nock';
import React from 'react';
import Weather from '../Weather';

const mockWeather = {
  summary: 'sunny',
};

describe('<Weather />', () => {
  beforeAll(() => {
    nock('https://weather.example.com/api')
      .get('/weather?q=london')
      .reply(200, mockWeather);
  });

  it('Component fetching weather from API', async (done) => {
    const root = shallow(<Weather location="london" />);

    let componentsWeather = {};
    await waitUntil(() => {
      componentsWeather = root.state('weather');
      return componentsWeather.summary !== undefined;
    });

    expect(componentsWeather.summary).toEqual(mockWeather.summary);

    done();
  });
});

```

As we see in the test above we have absolute control about what our endpoint returns. It is important that in your unit tests you will use a domain that doesn't exist in reality. In this case you can be certain that the unit tests isn't getting any data from the real internet. You can achieve this via configuration of your application.
