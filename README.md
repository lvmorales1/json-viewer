# JSON Viewer

A clean JSON viewer built with Vue 3 and Tailwind. Paste your JSON on the left, explore it on the right.

## What it does

- Renders JSON as a collapsible tree — objects and arrays can be opened and closed
- Highlights strings, numbers, booleans and nulls in different colors
- Lets you search across keys and values, with highlighted matches and next/prev navigation
- Handles large JSONs without freezing — search is debounced and big arrays are paginated (100 items at a time)
- Expand all / Collapse all buttons
- Copy the formatted JSON to clipboard
- Tree graph visualization of the JSON structure

## Running it locally

```bash
npm install
npm run dev
```

## Stack

- Vue 3 with `<script setup>`
- Vite
- Tailwind CSS v4
