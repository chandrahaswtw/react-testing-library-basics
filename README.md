# INTRODUCTION

NOTE: This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

### `npm test`

All the test file ends with .spec.js or .test.js or inside the \_\_test\_\_ folder, where it has simply the .js files inside are considered the test files and `jest` will pick up all these files.

# ALL ABOUT @testing-library/react

## Queries

### About Queries

- Single Elements
  - `getBy...` Returns the matching node for a query, and throw a descriptive error if no elements match or if more than one match is found
  - `queryBy...` Returns the matching node for a query, and return `null` if no elements match. This is useful for asserting an element that is not present.
  - `findBy...` Returns a Promise which resolves when an element is found which matches the given query. The promise is rejected if no element is found or if more than one element is found after a default timeout of 1000ms.
- Multiple Elements

  - `getAllBy...` Returns an array of all matching nodes for a query, and throws an error if no elements match.
  - `queryAllBy...` Returns an array of all matching nodes for a query, and return an empty array ([]) if no elements match.
  - `findAllBy...` Returns a promise which resolves to an array of elements when any elements are found which match the given query. The promise is rejected if no elements are found after a default timeout of 1000ms.

  ![Alt text](/images/QueryCheatSheet.png)

#### Summing up all

| Goal of test                       | Use                     | Comments                                                                   |
| :--------------------------------- | :---------------------- | :------------------------------------------------------------------------- |
| Prove an element exists            | `getBy`, `getAllBy`     |
| Prove an element does not exist    | `queryBy`, `queryAllBy` |
| Prove an element eventually exists | `findBy`, `findAllBy`   | This waits for 1 sec by default to find the element and if not, it rejects |

An example of these:

```
Say we have a textbox missing in JSX and we intended to test it.

it("Basic example getBy queryBy findBy", async () => {
  render(<UserList users={users}></UserList>);
  expect(() => screen.getByRole("textbox")).toThrow();
  expect(screen.queryByRole("textbox")).toBeNull();
  await expect(screen.findByRole("textbox")).rejects.toThrow();
});

Say we have a single textbox missing in JSX and we intended to test it.

it("Basic example getBy queryBy findBy", async () => {
  render(<UserList users={users}></UserList>);
  expect(screen.getByRole("textbox")).toBeInTheDocument();
  expect(screen.queryByRole("textbox")).toBeInTheDocument();
  await expect(screen.findByRole("textbox")).resolves.toBeInTheDocument();
});


```

### ByRole

**Always prefer to use ByRole and if you don't have an option, then go and use others.**

Refer to this [link](https://www.w3.org/TR/html-aria/#docconformance) to see all roles.

Some common roles are as below:

- `<a href="/"></a>` &#8594; link
- `<button></button>` &#8594; button
- `<footer></footer>` &#8594; contentinfo
- `<h1></h1>` &#8594; heading (h1, h2, h3, h4, h5 and h6)
- `<header></header>` &#8594; banner
- `<img src="" alt="" />` &#8594; img
- `<input type="text" name="" id="" />` &#8594; textbox
- `<input type="checkbox" name="" id="" />` &#8594; checkbox
- `<input type="radio" name="" id="" />` &#8594; radio
- `<input type="number" name="" id="" />` &#8594; spinbutton
- `<li></li>` &#8594; listitem
- `<ul></ul>` &#8594; list
- `thead` &#8594; rowgroup
- `tbody` &#8594; rowgroup
- `tr` &#8594; row
- `th` &#8594; columnheader
- `td` &#8594; cell


#### Example 1 : Basic use or role

```
const inputs = screen.getAllByRole("textbox");
const button = screen.getByRole("button");
```

#### Example 2 : Get a specific element based on name

If we want to be more specific we can with `getByRole` as below. The second parameter is the `name` which typically looks for the label text.

```
<button>submit</button>
```

```
screen.getByRole("button", {name : /submit/i });
```

##### Example 3 : Get a specific element based on label

We can selectively choose elements based on the label. Say for a textbox we have the below HTML

```
<label htmlFor="fName">Full Name</label>
<input id="fName" />
```

```
screen.getByRole("textbox", { name: /Full Name/i });
```

##### Example 4 : Get a specific element based on aria-label

Sometimes buttons don't have inner text, we can add `aria-label` to tell what the gutton actually does. Say for example we have a **search** button as below:

```
<button aria-label="search">
  <svg/>
</button>
```

```
screen.getByRole("button", {name : /search/i });
```

### ByLabelText

```
<label htmlFor="emailID">Email</label>
<input type="text" id="emailID" />
```

- Now to get the appropriate input element we do as. It specifically searches for label with text content `/email/i`.
- `ByLabelText` works only on form elements (like inputs, checkboxes, radio buttons) that are labeled using the label element.
- If we wish to use for other elements we can use `aria-label` and use `ByLabelText`

```
const nameInput = screen.getByLabelText(/email/i);
```

If we wanted to choose something particular but not any element, use the format discussed in `ByRole` with `name` as second parameter.

### ByText

If we don't care about what element but if we want to fetch element based on visible text on the DOM, we do the below:

```
const nameInput = screen.getByText("email");
```

**Note**: ByText looks for the entire text say we are looking for `30` but we have a `<div>30 items</div>` it fails as it looks for the entire text within a tag. In that case We can use regular expressions instead as `/30/` and this just looks for a matching text.

### ByDisplayValue

It is used to get the input textboxes based on value inside it.

Say for example:

```
<input type="text" id="lastName" />
document.getElementById('lastName').value = 'Norris'
```

We can test as below:

```
const lastNameInput = screen.getByDisplayValue('Norris')
```

We can do the same for textarea and select dropdowns as well.

### ByAltText

For search for all images based on alt text we do as below:

```
<img alt="Incredibles 2 Poster" src="/incredibles-2.png" />

const incrediblesPosterImg = screen.getByAltText("Incredibles 2 Poster")
```

### ByTitle

As we know we use `title` to show extra information about element, when we hover over as native tooltip HTMP provides, we can see the info. To do so, we do the below:

```
<span title="Delete" id="2"></span>
```

```
const deleteElement = screen.getByTitle('Delete')
```

### ByTestId

**This must be the last option, if nothing works choose this**

```
<div data-testid="custom-element" />
```

```
const element = screen.getByTestId('custom-element')
```

**Note: All the above we can use regular expressions instead of the strings we pass through. It's a choice of what we are looking in a test**

## within

If we want to search something within the found element we can use `within` which need to be imported from `@testing-library/react` as below:

```
import { within } from "@testing-library/react";
```

```
render(<UserList></UserList>);

const rows = within(screen.getByTestId("userListTesting")).getAllByRole("row");
expect(rows).toHaveLength(2);
```

In the above we did for `getByTestId``, we can use any query..

## Matchers

React testing library exposes extra matchers along with the ones that jest provides and are exposed on the global variable `expect`. We can find the whole list of matchers here -> https://github.com/testing-library/jest-dom#custom-matchers

### Custom matchers

Say we have to repeat a functionality and we want a custom matcher, say if we need to search for an element `within` we can do as below:

```
function toContainRole(container, role, quantity = 1) {
  const elements = within(container).queryAllByRole(role);
  if (elements.length === quantity) {
    return {
      pass: true,
    };
  } else {
    return {
      pass: false,
      message: () =>
        `Expected to find ${quantity} ${role} elements but got ${elements.length} elements`,
    };
  }
}
```

`expect` expects return value in a praticular way. Return value must be an object with key `pass` a boolean and/or `message` a function with a return string if we need to display in case the test fails.

Now We need to `extend` in `expect` as below:

```
  expect.extend({ toContainRole });
```

We can use normally in a test case as we does with expect as below:

```
render(<UserList></UserList>);
const targetNode = screen.getByTestId("userListTesting");
expect(targetNode).toContainRole("row", 2);
```

## Rerender the component

There might be a need where we may feel like to re-render the component and examine the output. For example, if we wish to change props and re-render, we can do a below:

```
const { rerender } = render(<TheComponent {...props} />);
props.TheKey = TheValue
rerender(<TheComponent {...props} />);
```

## Using direct DOM commands.

Sometimes we cannot find elements in any which way and we need to rely on DOM, we can do as below:

```
const { container } = render(<TheComponent {...props} />);
```
Now we can access the elements as:

```
const linkElement = container.querySelector("label > a");
```

### NOTE:
- Just imagine the `container` as `DOM - document` and we can run all commands we run on `document`.
- The `linkElement` we are getting from `container` is of same node structure we would've got from other react testing library commands discussed so far. We can do all assertions as we done earlier.

### Difference between the `container` apporach and other commands on `screen` (get, query, find etc)

This is a very important difference to keep in mind.

Example 1: See the below, when we use `container`, it gets hold of the DOM. Even if we `rerender`, we can use the same `container` variable, no need to refresh it as it's automatically pointing to the DOM.

```
const { rerender , container } = render(<TheComponent {...props} />);
expect(container.something).someChecking.....
rerender(<TheComponent {...props} />)
expect(container.something).someChecking.....
```

Example 2: See the below, the commands from `screen.*` need to be refreshed after rerender, else they will be pointing to the same old element. 

```
render(<TheComponent {...props} />);
let TheElement = screen.getBySomething("something")
rerender(<TheComponent {...props} />)
TheElement = screen.getBySomething("something")
```

## User interactions

We have fireEvent imported and used as below. We can find the documentation [here](https://testing-library.com/docs/dom-testing-library/api-events/). It offers a few use cases and is recommended to use `@testing-library/user-event`

```
import {fireEvent} from "@testing-library/react"
```



## Testing playground

If you add the below in your testcase and it generates an URL on console when we run the tests.

```
screen.logTestingPlaygroundURL();
```

Now we when we open the generated URL we get the UI as below, when we hover over any item we get the corresponding query there.

![Alt text](/images/testingPlayground1.png)

At times we cannot get the required query. Say we wanted to check for `tr` we cannot hover, just add some extra styles and we can get the exact query as below:

![Alt text](/images/testingPlayground2.png)

## Screen debug

The below prints all the resultant HTML on console, where we can check what is the DOM.

```
screen.debug()
```

# ALL ABOUT @testing-library/user-event

```
import user from "@testing-library/user-event";
```

We can simulate the user events as below:

- `user.click(element)` simulates clicking on provided element
- `user.keyboard('asdf')` simulates typing asdf. To type something we first need to click on the elemene first.
- `user.keyboard('{Enter}')` simulates pressing enter key.
- `user.type(element, someText)` simulates typing in a text box. We can also use the combination of `user.click(element)` and `user.keyboard('asdf')` but this provides a direct solution.
- `user.tab()` simulates pressing tab key.
# waitFor and act

## act warnings

`act` gives a small window which enables the state changes. The recommendations give by terminal to wrap with `act` are misleading. The message says you should! Don't do it. Do not add `act` to your test. Using the below, react testing library already wraps within the `act` for us internally.

- screen.findBy (Discussed above)
- screen.findAllBy (Discussed above)
- user events from `@testing-library/user-event`
- waitFor :

```
await waitFor(() => {
   We don't need to use find* here. We can directly call get* or query* to fetch the results directly as waitFor waits for us. We can use this whenever we are expecting a state change, or an API call etc.
})
```

## waitFor timeout

`waitFor` has a default timeout of `1000 ms`, we can increase the timeout as below:

```
await (waitFor(() => screen.getByText('something'),{timeout:3000}));
```

## Using waitFor with jest timers

Say we are introducing custom timeouts with jest timers, we can do on following ways

### Keeping jest timer inside waitFor()

We need to manually set the timeout within `waitFor` as we are running the timer within `waitFor` and it will timeout after it's default `1000ms`

```
await waitFor(
  () => {
    jest.advanceTimersByTime(10000);
    expect(screen.getByText('something')).toBeInTheDocument();
  }, {timeout:12000}
);
```

### Keeping jest timer outside waitFor() - Preferred way

We don't need to add the custom timeout for waitFor as we added the timer outside. This method is preferred.

```
jest.advanceTimersByTime(10000);
await waitFor(
  () => {
    expect(screen.getByText('something')).toBeInTheDocument();
  }
);
```


