# [react-final-form](https://final-form.org/docs/react-final-form/getting-started)

## :page_facing_up: 목차
- [Overview](#overview)
- [Usage](#usage)
- [Validation](#validation)
- [Subscription](#subscription)

## Overview

### Architecture
> React Final Form is a thin React wrapper for [Final Form](https://final-form.org/), which is a subscriptions-based form state management library that uses the Observer pattern, so only the components that need updating are re-rendered as the form's state changes.

> By default, **React Final Form subscribes to all changes**, but if you want to fine tune your form to optimized blazing-fast perfection, you may specify only the form state that you care about for rendering your gorgeous UI. You can think of it a little like GraphQL's feature of only fetching the data your component needs to render, and nothing else.

### Component
<img src="https://images.ctfassets.net/abqv6r6rj8r3/32jwn0w0TxLK3oiS0UvsJq/9a8df7ca10b91cb91a2deea82b208eb3/final-form-diagram.png?w=610" width="400" height="500"></img>

`<Form/>`: A component that surrounds your entire form and manages the form state. It can inject form state and functionality, e.g. a handleSubmit function for you to pass to your `<form>` element, via render props.

`<Field/>`: A component that lives inside your `<Form/>` and creates a "field". It register itself with the surrounding `<Form/>` tag and manages all the state for a particular field, providing input callbacks (e.g. onBlur, onChange, and onFocus) as well as the value of the form and myriad metadata about the state of the field.

`<FormSpy/>`: [Advanced Usage] A component that can tap into form-wide state from inside your `<Form/>`. It's primarily only for advanced usage when form renders are being restricted via the subscription prop.


## Usage

```jsx
import { Form, Field } from 'react-final-form'

const MyForm = () => (
  <Form
    onSubmit={onSubmit}
    validate={validate}
    render={({ handleSubmit }) => (
      <form onSubmit={handleSubmit}>
        <h2>Simple Default Input</h2>
        <div>
          <label>First Name</label>
          <Field name="firstName" component="input" placeholder="First Name" />
        </div>

        <h2>An Arbitrary Reusable Input Component</h2>
        <div>
          <label>Interests</label>
          <Field name="interests" component={InterestPicker} />
        </div>

        <h2>Render Function</h2>
        <Field
          name="bio"
          render={({ input, meta }) => (
            <div>
              <label>Bio</label>
              <textarea {...input} />
              {meta.touched && meta.error && <span>{meta.error}</span>}
            </div>
          )}
        />

        <h2>Render Function as Children</h2>
        <Field name="phone">
          {({ input, meta }) => (
            <div>
              <label>Phone</label>
              <input type="text" {...input} placeholder="Phone" />
              {meta.touched && meta.error && <span>{meta.error}</span>}
            </div>
          )}
        </Field>

        <button type="submit">Submit</button>
      </form>
    )}
  />
)
```
[https://codesandbox.io/s/github/sangkukbae/react-final-form](https://codesandbox.io/s/github/sangkukbae/react-final-form)

## Validation

```jsx
import React from "react";
import { Form, Field } from "react-final-form";
import Styles from "./Styles";

const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

const onSubmit = async values => {
  await sleep(300);
  window.alert(JSON.stringify(values, 0, 2));
};

const required = value => (value ? undefined : "Required");
const mustBeNumber = value => (isNaN(value) ? "Must be a number" : undefined);
const minValue = min => value =>
  isNaN(value) || value >= min ? undefined : `Should be greater than ${min}`;
const composeValidators = (...validators) => value =>
  validators.reduce((error, validator) => error || validator(value), undefined);

const FormComponent = () => (
  <Styles>
    <Form
      onSubmit={onSubmit}
      render={({ handleSubmit, form, submitting, pristine, values }) => (
        <form onSubmit={handleSubmit}>
          <Field name="firstName" validate={required}>
            {({ input, meta }) => (
              <div>
                <label>First Name</label>
                <input {...input} type="text" placeholder="First Name" />
                {meta.error && meta.touched && <span>{meta.error}</span>}
              </div>
            )}
          </Field>
          <Field name="lastName" validate={required}>
            {({ input, meta }) => (
              <div>
                <label>Last Name</label>
                <input {...input} type="text" placeholder="Last Name" />
                {meta.error && meta.touched && <span>{meta.error}</span>}
              </div>
            )}
          </Field>
          <Field
            name="age"
            validate={composeValidators(required, mustBeNumber, minValue(18))}
          >
            {({ input, meta }) => (
              <div>
                <label>Age</label>
                <input {...input} type="text" placeholder="Age" />
                {meta.error && meta.touched && <span>{meta.error}</span>}
              </div>
            )}
          </Field>
          <div className="buttons">
            <button type="submit" disabled={submitting}>
              Submit
            </button>
            <button
              type="button"
              onClick={form.reset}
              disabled={submitting || pristine}
            >
              Reset
            </button>
          </div>
          <pre>{JSON.stringify(values, 0, 2)}</pre>
        </form>
      )}
    />
  </Styles>
);

export default FormComponent;
```

[https://codesandbox.io/s/final-form-validation-g1dmb](https://codesandbox.io/s/final-form-validation-g1dmb)

## Subscription

```jsx
import React from "react";
import { Form, Field, FormSpy } from "react-final-form";
import RenderCount from "./RenderCount";
import Styles from "./Styles";

const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

const onSubmit = async values => {
  await sleep(300);
  window.alert(JSON.stringify(values, 0, 2));
};

const required = value => (value ? undefined : "Required");

const MyForm = ({ subscription }) => (
  <Styles>
    <Form
      onSubmit={onSubmit}
      subscription={subscription}
      render={({ handleSubmit, form, submitting, pristine, values }) => (
        <form onSubmit={handleSubmit}>
          <RenderCount />
          <Field name="firstName" validate={required}>
            {({ input, meta }) => (
              <div>
                <RenderCount />
                <label>First Name</label>
                <input {...input} placeholder="First Name" />
                {meta.touched && meta.error && <span>{meta.error}</span>}
              </div>
            )}
          </Field>
          <Field name="lastName" validate={required}>
            {({ input, meta }) => (
              <div>
                <RenderCount />
                <label>Last Name</label>
                <input {...input} placeholder="Last Name" />
                {meta.touched && meta.error && <span>{meta.error}</span>}
              </div>
            )}
          </Field>
          <div className="buttons">
            <button type="submit" disabled={submitting}>
              Submit
            </button>
            <button
              type="button"
              onClick={form.reset}
              disabled={submitting || pristine}
            >
              Reset
            </button>
          </div>
          {values ? (
            <pre>
              <RenderCount />
              {JSON.stringify(values, 0, 2)}
            </pre>
          ) : (
            <FormSpy subscription={{ values: true }}>
              {({ values }) => (
                <pre>
                  <RenderCount />
                  {JSON.stringify(values, 0, 2)}
                </pre>
              )}
            </FormSpy>
          )}
        </form>
      )}
    />
  </Styles>
);

export default MyForm;
```

[https://codesandbox.io/s/final-form-subscription-zhmen](https://codesandbox.io/s/final-form-subscription-zhmen)

:memo: **참고 자료**   
[https://final-form.org/docs/react-final-form/getting-started](https://final-form.org/docs/react-final-form/getting-started)   
[https://codeburst.io/final-form-the-road-to-the-checkered-flag-cd9b75c25fe](https://codeburst.io/final-form-the-road-to-the-checkered-flag-cd9b75c25fe)   
[https://medium.com/@erikras/final-form-arrays-and-mutators-13159cb7d285](https://medium.com/@erikras/final-form-arrays-and-mutators-13159cb7d285)   
[https://medium.com/@erikras/react-final-form-html5-validation-7055b867ac51](https://medium.com/@erikras/react-final-form-html5-validation-7055b867ac51)   
[https://www.basefactor.com/react-final-form-validation-fonk](https://www.basefactor.com/react-final-form-validation-fonk)   






