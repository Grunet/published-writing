# Bringing Together Keyboard-only and Click-based UI Tests with keyboard-testing-library

## Recap: Keyboard-focused Unit Testing

As covered in Marcy Sutton’s Testing Accessibility workshops, it’s impactful to write UI tests from the perspective of users who only use the keyboard.

You can do so with the Testing Library tools you’re likely already familiar with, as shown in this example lifted from the course:

```js
it('can be operated with the keyboard and assistive tech', async () => { let clicked = false render(<IconButton name="Fling it" onClick={()=> { clicked = true }} />) const button = screen.getByRole('button') await userEvent.tab() await userEvent.keyboard('[Enter]') expect(clicked).toBe(true) })

it('can be operated with the keyboard and assistive tech', async () => {
    let clicked = false 
    render(<IconButton name="Fling it" onClick={()=> { clicked = true }} />)
    const button = screen.getByRole('button')
    await userEvent.tab()
    await userEvent.keyboard('[Enter]')
    expect(clicked).toBe(true)
})
```

Adding more tests like this will help ensure your UI remains keyboard accessible as it continues to evolve.
But What About My Existing Click-based Tests?

Most of your existing tests will already be interacting with elements via userEvent.click. These tests may end up duplicating some of the setup and assertion code from your keyboard-only tests.

Over time, this could make the tests more difficult to maintain and extend.

## So How Can I Cut Down on the Duplication?

One idea would be to factor out those duplicated pieces into shared code. But that could also make the tests harder to read and reason about.

A different idea comes from noticing that tests will mostly come in pairs. For every test like

Fill out the form, then find and click the submit button

there will likely be another test like

Fill out the form, then navigate to the submit button using the keyboard and press Enter

Based on this, a solution could be to:

    Consolidate each matching pair of tests into a single, click-based test
    Create a way to automatically replace clicks with equivalent keyboard actions (without modifying test code)
    Run the test suite once in “click-based mode” and again in “keyboard-based mode”

That way, the setup and assertion code is only written once.

## A “Keyboard Testing Library” To Tackle The Problem

Trying to implement that idea is how the keyboard-testing-library package came to be.

### How It Works in Theory:

The navigateToAndPressEnter method on the library’s sole export ( keyboardOnlyUserEvent) works as follows:

Starting from the currently focused element, it performs a depth-first search through the DOM’s keyboard accessible elements. It uses Testing Library’s shims for pressing the tab, shift+tab, and arrow keys to systematically move between these elements.

If it finds the target element, it will stop and activate it. It uses Testing Library’s shim for pressing the Enter key to do so.

If it can’t find the target element, it will throw an error.

### How it Works in Reality

Rewriting the example from the introduction, the calls to userEvent.tab and userEvent.keyboard can be replaced with one call to keyboardOnlyUserEvent.navigateToAndPressEnter:

```js
it('can be operated with the keyboard and assistive tech', async () => { let clicked = false render(<IconButton name="Fling it" onClick={()=> { clicked = true }} />) const button = screen.getByRole('button') // await userEvent.tab() // await userEvent.keyboard('[Enter]') await keyboardOnlyUserEvent.navigateToAndPressEnter(button) expect(clicked).toBe(true) })

it('can be operated with the keyboard and assistive tech', async () => {
  let clicked = false
  render(<IconButton name="Fling it" onClick={()=> { clicked = true }} />)
  const button = screen.getByRole('button')
  // await userEvent.tab()
  // await userEvent.keyboard('[Enter]')
  await keyboardOnlyUserEvent.navigateToAndPressEnter(button)
  expect(clicked).toBe(true)
})
```

To enable switching between being click-based and keyboard-based, keyboardOnlyUserEvent.navigateToAndPressEnter can be conditionally replaced with userEvent.click depending on the presence of an environment variable:

```js
if (process.env["USE_KEYBOARD"]) { jest.spyOn(userEvent, "click") .mockImplementation(keyboardOnlyUserEvent.navigateToAndPressEnter); } it('can click the button', async () => { let clicked = false render(<IconButton name="Fling it" onClick={()=> { clicked = true }} />) const button = screen.getByRole('button') // await keyboardOnlyUserEvent.navigateToAndPressEnter(button) await userEvent.click(button) expect(clicked).toBe(true) })

if (process.env["USE_KEYBOARD"]) {
  jest.spyOn(userEvent, "click")
      .mockImplementation(keyboardOnlyUserEvent.navigateToAndPressEnter);
}

it('can click the button', async () => {
  let clicked = false
  render(<IconButton name="Fling it" onClick={()=> { clicked = true }} />)
  const button = screen.getByRole('button')
  // await keyboardOnlyUserEvent.navigateToAndPressEnter(button)
  await userEvent.click(button)
  expect(clicked).toBe(true)
})
```

This enables the test suite to run once without the environment variable set, executing the click-based versions of the tests. When it’s run again with the environment variable set, the keyboard-based versions of the tests will be executed.

Now every time someone writes a new click-based test, it will automatically be transformed into a keyboard-based test as well.

## A Big Caveat

A huge caveat is that I have only gotten to try this out on some toy examples so far. So it’s still unclear to me if it will work smoothly in most cases.

With that in mind, my guess is that it will be best suited to component library tests, and least well suited to full page tests.

## Backstory and Thanks

I originally thought of this idea at some point after hearing Marcy share her approach to the topic (now covered in the Testing Accessibility workshop series). Many thanks to her for that!

And thank you dear reader for coming this far! I hope this has been a helpful use of your time and spoons.