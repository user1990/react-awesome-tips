# react-awesome-tips

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

- Doesn't work well with defaultProps (Note: JS default args should be preferred anyway)

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
table-header.tsx;
```

**Why**

- Readable file names. e.g MyHalfFixedDedupedDirResolver vs my-half-fixed-deduped-dir-resolver 👀
- No more weird git conflicts when renaming/deleting/adding files on various OS file systems (case-sensitive/insensitive)
- Consistency (I don’t have to think if this file is component or some helper or service. tsx extension tells me that)
- Nicely maps to component implementation name table-header.tsx 👉 `const TableHeader = () => {}`
