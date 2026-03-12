---
name: website-design
description: How to design a website using NextJS + Tailwind, and how to create a new front-end project. How design a website as a professional developer. Use when develop a website
---

## When initializing the project:

The technology stack chosen is: nextjs@16, tailwindcss@4

## Website design rules

1. Add the class name "cursor-pointer" to all clickable elements.

2. For the initial page data, request it in layout.tsx and pass it to child components via the provider. Each layout.tsx must be a React Server component.

3. If your project uses a UI component library, prioritize using the existing components, and then use Tailwind CSS.

4. The total number of font sizes in the entire project should not exceed 5; the number of font sizes should not be increased arbitrarily.

5. The website uses the following colors.

```
Primary: #4F46E5 (Indigo, for buttons and links)

Primary Hover: #3730A3 (Deep indigo, for interactions)

Success: #10B981 (Mint green, for confirmations)

Warning: #F59E0B (Amber yellow, for alerts)

Error: #EF4444 (Coral red, for errors)

Background: #F8FAFC (Light gray-blue, page base)

Surface: #FFFFFF (White, cards and content)

Text Primary: #1E293B (Dark slate, body text)

Text Secondary: #64748B (Medium gray, descriptions)

Border: #E2E8F0 (Soft gray, dividers)
```

