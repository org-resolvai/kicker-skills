---
name: translate
description: Translate text accurately while preserving formatting, placeholders, and tone for the target locale.
license: MIT-0
---

# Translate

Master accurate translation that preserves meaning, formatting, and cultural context.

## Formatting Preservation

- Never translate content inside code blocks, HTML tags, or markdown syntax.
- Preserve placeholders like `{name}`, `{{variable}}`, `%s`, and `$1` exactly as-is.
- Keep markdown structure intact, including headers, links, and bold or italic formatting.
- Maintain JSON/XML structure and keys. Translate only values where appropriate.

## Content Rules

- Do not translate proper nouns, brand names, URLs, email addresses, or technical terms that should remain unchanged.
- Do not translate code snippets, CSS classes, API endpoints, or file extensions.
- Preserve numbers, dates, and IDs in their original format unless locale conversion is explicitly needed.
- Keep terminology consistent throughout the full translation.

## Language-Specific Handling

- Use plural forms that match the target language instead of copying English patterns.
- Preserve correct gender agreement in gendered languages.
- Choose the right formality level for the audience and locale.
- For RTL languages, account for text direction while keeping URLs and similar LTR elements stable.

## Cultural Adaptation

- Adapt idioms and expressions for meaning instead of translating them literally.
- Convert units when culturally appropriate.
- Adjust date formats to locale conventions when needed.
- Use local currency symbols and number formatting where appropriate.

## Context Awareness

- Disambiguate terms based on context.
- Keep the original tone, intent, and audience fit.
- Preserve emotional nuance, not just literal wording.

## Quality Control

- Read the full context before translating.
- Check that the result sounds natural in the target language.
- Verify that formatting and placeholders still work after translation.
- Ensure terminology and voice stay consistent across the whole document.
