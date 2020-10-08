# React Awesome code guide tips

Whole article is written like an “style guide” with 3 sub-sections for every tip/pattern which consists of:

- **Don’t** 🚨 ( code example what you shouldn’t be doing)

- **Do** ✅ or Good/Better/Consider (code example what you should be doing)

- **Why** 🧐 (reasoning/explanation)

---

## Don't use React.FC in Typescript Component

**Don't**

```javascript
export const Card: React.FC<CardProps> = ({ event }) => {
```

**Do**

```javascript
export const Card = ({ event }: CardProps) => {
```

**Why**

- Implicit optional children - better define your props API explicitly

- No generic component creation possible

- Is removed from `create-react-app` boilerplate, check more [here](https://github.com/facebook/create-react-app/pull/8177)

- No need to have explicit ReactElement | null return type, which in addition is not accurate; it can be 1) better inferred automatically by TS or 2) set - explicitely

- FC is just a wrapper for statics like contextTypes, propTypes, etc. which aren't needed in most cases

- Doesn't work well with defaultProps (Note: JS defaul-t args should be preferred anyway)

- No need to remember "magic" types - just use functions for components

- Doesn't make your code (much) leaner

---

## Don’t use camelCase/PascalCase for file names

**Don't**

```javascript
TableHeader.tsx;
```

**Do**

```javascript
table - header.tsx;
```

**Why**

- Readable file names. e.g MyHalfFixedDedupedDirResolver vs my-half-fixed-deduped-dir-resolver 👀
- No more weird git conflicts when renaming/deleting/adding files on various OS file systems (case-sensitive/insensitive)
- Consistency (I don’t have to think if this file is component or some helper or service. tsx extension tells me that)
- Nicely maps to component implementation name table-header.tsx 👉 `const TableHeader = () => {}`

---

## Don’t use return statement for dumb component

**Don't**

```javascript
export const CardList = ({ item }: CardListProps) => {
  return <CardContainer>SOME JSX</CardContainer>;
};
```

**Do**

```javascript
export const CardList = ({ item }: CardListProps) => (
  <CardContainer>SOME JSX</CardContainer>
);
```

**Why**

- When component don't have logic like hooks usage or declared variables, so the curly brackets and return statement is unnecessary
- Shorter and cleaner syntax

---

## Don’t use JSX.Element to annotate function/component return type or children/props

**Don't**

```javascript
type SomeProps {
  children: JSX.Element;
}
```

**Do**

```javascript
import React, { ReactChild, ReactChildren } from "react";

interface SomeProps {
  children: ReactChild | ReactChildren;
}
```

**Why**

- globals are bad ☝️💥
- TypeScript supports locally scoped JSX to be able to support various JSX factory types and proper JSX type checking per factory. While current react types use still global JSX namespace, it’s gonna change in the future.
- explicit types over generalized ones

---

## React Fragment

**Don't**

```javascript
return (
  <div>
    <ChildA />
    <ChildB />
    <ChildC />
  </div>
);
```

**Do**

```javascript
return (
  <>
    <ChildA />
    <ChildB />
    <ChildC />
  </>
);
```

Then mapping data Fragment can take a key parameter

```javascript
return (
  <Item>
    {props.items.map((item) => (
      <React.Fragment key={item.id}>
        <ItemText>{item.term}</ItemText>
        <ItemDescription>{item.description}</ItemDescription>
      </React.Fragment>
    ))}
  </Item>
);
```

**Why**

- It’s a tiny bit faster and has less memory usage (no need to create an extra DOM node). This only has a real benefit on very large and/or deep trees, but application performance often suffers from death by a thousand cuts. This is one cut less.
- Some CSS mechanisms like Flexbox and CSS Grid have a special parent-child relationship, and adding divs in the middle makes it hard to keep the desired layout while extracting logical components.
- The DOM inspector is less cluttered.
- Better semantic jsx markup.

---

## Use double negation on Conditional Rendering with &&

**Don't**

```javascript
const App = ({ items }) => (
  <>
    {items.length &&
      items.map((item) => <div key={item.label}>{item.label}</div>)}
  </>
);
```

**Do**

```javascript
const App = ({ items }) => (
  <>
    {!!items.length &&
      items.map((item) => <div key={item.label}>{item.label}</div>)}
  </>
);
```

**Why**

- This will actually render a number 0 on the screen when items.length is empty. JavaScript considers the number 0 as a falsey value, so when items is an empty array, the && operator won't be evaluating the expression to the right of it, and will just return the first value.
- That way, if items is an empty array, react will not render anything on the screen if the evaluated output is a boolean.

---

## Use `Styled` prefix for all styled components & their types

**Don't**

```javascript
type ElementProps {
  hex: string;
  imageUrl: string;
}

const Element = styled.div`
 ${(props: EventHeroImageProps) =>
    `background: linear-gradient(90deg, ${props.hex} -100%, rgba(255, 255, 255, 0) 100%), url(${props.image});`}
  width: 100%;
  height: 100%;
`;
```

**Do**

```javascript
type StyledElementProps {
  hex: string;
  imageUrl: string;
}

const StyledElement = styled.div`
 ${(props: EventHeroImageProps) =>
    `background: linear-gradient(90deg, ${props.hex} -100%, rgba(255, 255, 255, 0) 100%), url(${props.image});`}
  width: 100%;
  height: 100%;
`;
```

**Why**

- That way is easier to understand what component is in JSX
- Less messed up html elements

---

## Don't use Redux connect, instead use Redux Hooks `useSelector` and `useDispatch`.

**Don't**

```javascript
import React from "react";
import { Dispatch } from "redux";
import { connect } from "react-redux";
import ProductItem from "../components/ProductItem";
import { AppStore, Product } from "../types";
import { actions } from "../actions/constants";

const mapStateToProps = (state: AppStore) => ({
  products: state.productsOldModule.products,
});

const mapDispatchToProps = (dispatch: Dispatch) => {
  return {
    addItem: (product: Product) =>
      dispatch({ type: actions.ADD_TO_CART_OLD, payload: product }),
  };
};

const ProductListContainer = (props: Props) => {
  const { products } = props;
  return (
    <>
      {products.map((product: Product) => {
        return (
          <ProductItem
            product={product}
            key={product.id}
            dispatchToStore={props.addItem}
          />
        );
      })}
    </>
  );
};

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(ProductListContainer);
```

**Do**

```javascript
import React from "react";
import { useSelector, useDispatch } from "react-redux";
import { AppStore, Product } from "../types";
import ProductItem from "../components/ProductItem";
import { actions } from "../actions/constants";

const ProductListHooks = () => {
  const dispatch = useDispatch();
  const products: Product[] = useSelector(
    (state: AppStore) => state.productsModule.products
  );

  return (
    <>
      {products.map((product: Product) => {
        return (
          <ProductItem
            product={product}
            key={product.id}
            dispatchToStore={dispatch}
          />
        );
      })}
    </>
  );
};

export default ProductListHooks;
```

**Why**

- No more connect HOC’s so less nodes in our component hierarchy.
- Less boilerplate
- Could abstract our selectors for more robust and reusable code.
- More readable in a sense.
- Less separation of concerns

**Note**: One downside is the additional need of useMemo/useCallback for performance optimization

---

## Use `Styled` prefix for all styled components & their types

**Don't**

```javascript
type ElementProps {
  hex: string;
  imageUrl: string;
}

const Element = styled.div`
  ${(props: ElementProps) => {
    return `background: linear-gradient(90deg, ${hex} -100%, rgba(255, 255, 255, 0) 100%), url(${imageUrl});`;
  }}
  width: 100%;
  height: 100%;
`;
```

**Do**

```javascript
type StyledElementProps {
  hex: string;
  imageUrl: string;
}

const StyledElement = styled.div`
  ${(props: StyledElementProps) => {
    return `background: linear-gradient(90deg, ${hex} -100%, rgba(255, 255, 255, 0) 100%), url(${imageUrl});`;
  }}
  width: 100%;
  height: 100%;
`;
```

**Why**

- That way is easier to understand what component is in JSX
- Less messed up html elements

---

## Compose styles in Styled components

**Don't**

```javascript
const StyledMessage = styled.div`
  display: flex;
  margin: 0 auto;
  flex-direction: row;
  align-items: center;
  justify-content: left;
  text-align: left;
`;

const StyledMessageSuccess = styled.div`
  display: flex;
  margin: 0 auto;
  flex-direction: row;
  align-items: center;
  justify-content: left;
  text-align: left;
  background: green;
  border-color: #000;
`;

const StyledMessageDanger = styled.div`
  display: flex;
  margin: 0 auto;
  flex-direction: row;
  align-items: center;
  justify-content: left;
  text-align: left;
  background: red;
  border-color: #fff;
`;
```

**Do**

```javascript
const StyledMessage = styled.div`
  display: flex;
  margin: 0 auto;
  flex-direction: row;
  align-items: center;
  justify-content: left;
  text-align: left;
`;

const StyledMessageSuccess = styled(StyledMessage)`
  background: green;
  border-color: #000;
`;

const StyledMessageDanger = styled(StyledMessage)`
  background: red;
  border-color: #fff;
`;
```

**Why**

- Extend main component into several new ones
- Reduce repetitive code
- Pass any component to existing one, not just DOM elements

---

## Pitfalls with the useState React hook

**Don't**

```javascript
const [user, setUser] = useState({
  name: "John",
  email: "john@example.com",
  age: 25,
});
```

**Do**

```javascript
const [name, setName] = useState("John");
const [email, setEmail] = useState("john@example.com");
const [age, setAge] = useState(25);
```

**Why**

- Unexpected behavior
- Avoiding deep cloning
- Pass any component to existing one, not just DOM elements
