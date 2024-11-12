-   Feature Name: Text Formatting Extensions for Revolt Standard Markdown
-   Start Date: (29/08/2024)
-   RFC PR: https://github.com/revoltchat/rfcs/issues/0000
-   Tracking Issue: https://github.com/revoltchat/revolt/issues/0000
-   Status: draft

# Summary

This RFC extends RSM, that is Revolt Standard Markdown, to include text formatting extensions. This will allow users to format text in a more expressive way.

# Motivation

Text formatting is an essential part of communication. The expression using varying colour, size, and typeface can help convey the tone and emotion of the message.  
While rudimentary text formatting can be achieved by abusing the existing math syntax using KaTeX, it is neither user-friendly, nor ergonomic, nor intuitive, nor the intended use of the math syntax.

KaTeX is a math rendering engine, and not only is it not designed for general-purpose typesetting (like LaTeX is), but it also has not been demonstrably feasible to implement inline KaTeX rendering in mobile clients, which is a significant portion of Revolt's userbase, or desktop clients, which may be an area of future development.

In fact, as KaTeX is a JavaScript library that outputs HTML which itself depends on the correct rendering of the KaTeX CSS, the only way to render KaTeX in a client is to use a so-called "webview" in which a browser window embedded in the client. This is not only a performance concern, but also in any case a security liability and a potential security concern as such.

Even a theoretical perfect implementation of KaTeX rendering in a client would not be able to render KaTeX in a way that is consistent with the rest of the client's UI, as KaTeX is not designed to be styled in the same way as the rest of the client's UI. This would result in a jarring and inconsistent user experience. For instance, the used typefaces are in every case KaTeX-specific typefaces, causing formatted text conceived in such a way to stand out immediately to trained eyes.

A dedicated text formatting extension would allow for a more user-friendly, ergonomic, and intuitive way to format text, and would also allow for more flexibility in the future, as this extension is designed to be simple to amend if necessary. However, client-specific extensions are explicitly _not_ allowed as a result of this RFC, as this would defeat the purpose of RSM. Any client implementing such an extension would be considered non-compliant with RSM as a result.

Additionally, a dedicated text formatting extension would allow for a more consistent user experience not only across clients by virtue of being rendered at all, but also within a previously implementing client, as the text formatting extension would be styled in the same way as the rest of the client's UI.

# Guide-level explanation

Previously, you could only use advanced text formatting in Revolt using the following syntax:

```markdown
$$(\textcolor{red}{\textsf{This text is red}})$$
```

You can use advanced text formatting in Revolt by using the following syntax:

```markdown
{color: red}(This text is red)
{opacity: 50%; font: cursive}(This text is half-transparent and in cursive)
```

This means that not only are you now able to view coloured and styled text on Android and iOS, but you also have a more intuitive and user-friendly way to do so.

To include math formulas in your messages, you can continue using this math syntax:

```markdown
$$
\sqrt{3x-1}+(1+x)^2
$$
```

<!-- Explain the proposal as if it's already in Revolt and you were teaching it to new users.
- Introduce new concepts
- Explain the feature with examples
- What this fixes or adds and what users should think of the feature
- Discuss how this impacts using Revolt, how it makes it harder or easier to use

For internal oriented RFCs such as internal code changes, this should largely talk about how contributors should think about the change and give examples on the impacts. -->

# Reference-level explanation

The text formatting extension is a simple extension to RSM that allows for the formatting of text in a more expressive way. It is designed to be simple to implement and amend, and is designed to be consistent with the rest of the client's UI as well as allow for adjustments between platforms.

The text formatting extension is implemented as a double-brace syntax, with the first brace (STYLE BRACE), denoted by curly braces, wherein `{` is the left brace and `}` is the right brace, containing the formatting options. Immediately following without any whitespace is the second brace (CONTENT BRACE), denoted by parentheses, wherein `(` is the left brace and `)` is the right brace, containing the text to be formatted. The text to be formatted is allowed to contain any characters and is not subject to special restrictions.

```markdown
{STYLE BRACE}(CONTENT BRACE)
```

The STYLE BRACE contains the formatting options, which are separated by semicolons. The formatting options are key-value pairs, with the key and value separated by a colon. The key is the name of the formatting option, and the value is the value of the formatting option. The key is **case-insensitive**, and the value is **case-sensitive**. The key and value are separated by a colon, and the key-value pairs are separated by semicolons. The key is not allowed to contain any characters other than letters and the spe
, and the value is allowed to contain any characters with the exception of semicolons and closing braces. The key is not allowed to be empty, and the value is not allowed to be empty. The key is not allowed to be repeated. Keys that do not exist are ignored silently in formatting, but clients may choose to display a warning to the user sending the message, or to the user receiving the message, or both. Trailing semicolons are not allowed in the STYLE BRACE. The STYLE BRACE is not allowed to be empty.

We can deduce the following [grammar](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form) for the STYLE BRACE:

```bnf
STYLE_BRACE ::= "{" STYLE_PAIR "}"
STYLE_PAIR ::= STYLE_PAIR ";" STYLE_PAIR | STYLE_PAIR
STYLE_PAIR ::= KEY ":" VALUE
KEY ::= [a-zA-Z]+
VALUE ::= [^;]+
```

The following are examples of valid STYLE BRACEs:

```markdown
{color: red} // valid; sets the text color to red
{opacity: 50%; font: cursive} // valid; sets the text opacity to 50% and the font to cursive
{i-do-not: exist} // valid; ignored silently, as the key does not exist
{opacity: banana} // valid; ignored silently, as the value is not valid (banana is not permitted for the opacity key)
```

The following are examples of invalid STYLE BRACEs:

```markdown
{} // invalid; the key-value pair is empty and thus redundant
{color: red; color: blue} // invalid; the key is repeated
{color: red, opacity: 50%} // invalid; the key-value pairs are separated by commas instead of semicolons
{look&feel: red} // invalid; the key contains a character that is not a letter or a space
{color: red;} // invalid; trailing semicolon is not allowed
```

The following are examples of valid CONTENT BRACEs:

```markdown
{color: red}(This text is red) // valid; the text is "This text is red" and is red
{opacity: 50%; font: cursive}() // valid; the text is empty (note that we still format the void, as we do not care about the content); the void is half-transparent and in cursive
{color: red}(This text is red; the semicolon is part of the text) // valid; the text is "This text is red; the semicolon is part of the text" and is red
```

The following are examples of invalid CONTENT BRACEs:

```markdown
{color: red} // invalid; there is no CONTENT BRACE
```

## Standard value types

This RFC immediately introduces the following standard value types:

| Value type    | Description                               | Specification                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `color`       | Type used for colour keys                 | The value is a valid CSS colour. Includes gradients.                                                                                                                                                                                                                                                                                                                                                                                          |
| `solid-color` | Type used for solid colour keys           | The value is a valid CSS colour. Does not include gradients. Reserved for future use so clients can polish their implementation of non gradient colour parsing                                                                                                                                                                                                                                                                                |
| `percentage`  | Type used for percentage keys             | The value is a valid percentage. This means that the value is a number from and including 0 to and including 100, followed by a percent sign (`%`). No whitespace is allowed between the number and the percent sign.                                                                                                                                                                                                                         |
| `font`        | Type used for font family (typeface) keys | The value is one of `sans-serif`; `serif`; `monospace`; `cursive`; `casual`; `rounded`                                                                                                                                                                                                                                                                                                                                                        |
| `font-size`   | Type used for font size keys              | The value is a decimal number ranging from and including 0.25 to and including 5.00. Decimal points are optional, if omitted, the value is assumed to be an integer. If one decimal piece is present the value is assumed to be in tenths, if two decimal pieces are present the value is assumed to be in hundredths. Further decimal pieces can be truncated, rounded, ignored or used precisely, depending on the client's implementation. |
| `font-weight` | Type used for font weight keys            | The value is an integer in the range from and including 50 to and including 950 in no particular increments. Variable font weights are supported.                                                                                                                                                                                                                                                                                             |

Further value types may be published by Revolt in the future, and clients must implement them as they are published by Revolt to remain compliant with RSM.  
Additionally, changes to those value types may be published by Revolt in the future, and clients must implement them as they are published by Revolt to remain compliant with RSM.

Additionally, clients may publish or use their own value types, but for RSM compliance they may only be used in:

-   a server that has explicitly opted in to using the client's value types using a server-side configuration mechanism introduced with a future RFC (if such a mechanism is introduced), and
-   the saved notes channel

## Standard keys

This RFC immediately introduces the following standard keys.

Further keys may be published by Revolt in the future, and clients must implement them as they are published by Revolt to remain compliant with RSM.
Additionally, changes to those keys may be published by Revolt in the future, and clients must implement them as they are published by Revolt to remain compliant with RSM.

Additionally, clients may publish or use their own keys, but for RSM compliance they may only be used in:

-   a server that has explicitly opted in to using the client's keys using a server-side configuration mechanism introduced with a future RFC (if such a mechanism is introduced), and
-   the saved notes channel

### `color` and its alias `colour`

**Value type**: `color`  
**Description**: Sets the text colour to the specified colour. If `colour` is used instead of `color`, the key is still valid, but it must be ensured that only one of the two keys is used in a single STYLE BRACE.  
**Example**:

```markdown
{color: red}(This text is red)
{colour: linear-gradient(to right, red, orange)}(This text is a gradient from red to orange)
```

In HTML, this may yield the following:

```html
<span style="color: red;">This text is red</span>
<span style="color: linear-gradient(to right, red, orange);"
    >This text is a gradient from red to orange</span
>
```

### `opacity`

**Value type**: `percentage`  
**Description**: Sets the text opacity to the specified percentage.  
**Example**:

```markdown
{opacity: 50%}(This text is half-transparent)
```

In HTML, this may yield the following:

```html
<span style="opacity: 50%;">This text is half-transparent</span>
```

### `font`

**Value type**: `font`  
**Description**: Sets the text font to the specified font.  
**Notes**: If on web, you MUST NOT simply relay the value to the `font-family` CSS property. Instead, you MUST map the font to the appropriate `font-family` CSS property. Additionally, you MUST map the font to a font family that is either guaranteed to be installed (for instance, all iOS devices have the "New York" font installed) or load an appropriate font from a font provider (or the client's asset store).  
**Example**:

```markdown
{font: serif}(This text is in a serif font)
```

In HTML, this may yield the following:

```html
<!-- In our CSS, we load the "Roboto Serif" font from Google Fonts as an example. If we were to load a different font, we would change the font family accordingly. We use a fallback here but HTML output is neither specified nor shown as authoritative in this RFC. -->
<span style="font-family: 'Roboto Serif', serif;"
    >This text is in a serif font</span
>
```

### `size`

**Value type**: `font-size`  
**Description**: Multiplies the current font size by the specified factor.  
**Notes**: You MUST NOT simply relay the value to the `font-size` CSS property, for example by appending the `em` unit to the value. Instead, you MUST verify that the value is within the allowed range and then you may use the `em` unit to set the font size.  
**Example**:

```markdown
{font-size: 1.5}(This text is 1.5 times the size of the surrounding text)
```

In HTML, this may yield the following:

```html
<span style="font-size: 1.5em;"
    >This text is 1.5 times the size of the surrounding text</span
>
```

### `background`

**Value type**: `color`  
**Description**: Sets the background colour of the text.  
**Example**:

```markdown
{background: red}(This text has a red background)
```

In HTML, this may yield the following:

```html
<span style="background: red;">This text has a red background</span>
```

### `weight`

**Value type**: `font-weight`  
**Description**: Sets the font weight of the text.  
**Example**:

```markdown
{weight: 100}(This text is very light)
```

In HTML, this may yield the following:

```html
<span style="font-weight: 100;">This text is very light</span>
```


## Stacked Markup

The contents of the CONTENT BRACE may contain any Markdown (RSM) syntax. It will be parsed and rendered the same as if it were outside of the CONTENT BRACE, except that the text will be formatted according to the STYLE BRACE. In case default styles style an element (say, a [link](#)), the default styles of that element prevail over the styles specified in the STYLE BRACE.

This means that the following:

```markdown
{color: red}(This text is **red**)
```

will yield the following:

```html
<span style="color: red;">This text is <strong>red</strong></span>
```

but in the following:

```markdown
{color: lime}(This text is [a link](#))
```

the link will be, for example, underlined and the user's accent colour. This is because the default styles of the link element prevail over the styles specified in the STYLE BRACE.

However, the following:

```markdown
[{color: lime}(This text is a link)](#)
```

will yield a link with the text "This text is a link" in lime, because the link is styled within.

## Escaping

All standard Markdown (RSM) escaping rules apply to the left and right brace components of the text formatting extension.

## Deprecation of KaTeX inline rendering

As a result of this RFC, KaTeX inline rendering is deprecated and may or may not be removed in a future version of Revolt. Especially as it cannot be ported to non-web clients or non-JS clients without porting the entire KaTeX library, which is not feasible, and as it is not designed for general-purpose typesetting, it is not recommended to use KaTeX inline rendering for text formatting.  
**Block-level KaTeX rendering is not affected by this RFC and is not deprecated! It is still the recommended way to render math formulas in Revolt.**

Example of block-level KaTeX rendering:

```markdown
$$
\sqrt{3x-1}+(1+x)^2
$$
```

Yields:

$$
\sqrt{3x-1}+(1+x)^2
$$

You can continue using this syntax to render math formulas in Revolt.

### What about people using inline math in their messages?

We would have loved to keep both inline math and text formatting extensions, but as KaTeX inline rendering is not feasible to implement in non-web clients, we have to make a choice. This RFC has chosen to deprecate KaTeX inline rendering in favour of text formatting extensions, as text formatting extensions are more user-friendly, ergonomic, and intuitive, and allow for a more consistent user experience across clients, while keeping block-level KaTeX rendering for math formulas. **Not deprecating inline KaTeX rendering would not lead it to suddenly be feasible to implement in non-web clients.**

<!-- This is the technical section of the RFC, it should go over in detail:
- Its interaction with other features
- How this will be implemented
- Corner or edge cases

This section should reference the examples in the previous section and disect them in more detail. -->

# Drawbacks

There may be accessibility concerns around the use of text formatting extensions.
Those could include, with the corresponding mitigations:

-   **Colour contrast**: The text colour may not have enough contrast with the background colour, making it hard to read for some users. This can be mitigated by ensuring that the text colour has enough contrast with the background colour, for example by allowing the user to specify a maximum saturation for text coloured by the syntax described in this RFC. Alternatively, an accessibility option may automatically add a background colour to the text that is the inverse of the text colour, ensuring that the text is always readable. However this is up to the client to implement and decide. It is not a requirement of this RFC.
-   **Font size**: The font size may be too small or too large for some users. This can be mitigated by allowing the user to specify a maximum font size for text sized by the syntax described in this RFC and a minimum font size for text sized by the syntax described in this RFC. However this is up to the client to implement and decide. It is not a requirement of this RFC.

Additionally there may be concerns around the deprecation of KaTeX inline rendering, as some users may have gotten used to it. However, as KaTeX inline rendering is not feasible to implement in non-web clients, it is not considered a choice to keep it. **Not deprecating inline KaTeX rendering would not lead it to suddenly be feasible to implement in non-web clients.**.

# Rationale and alternatives

-   Are there alternative ways to solve this

Probably about a million.

-   Could this be done with existing features or existing solutions

No, Markdown does not support text formatting extensions by default. The deprecated KaTeX inline rendering is not a good fit for text formatting extensions, as it is not designed for general-purpose typesetting and is not feasible to implement in non-web clients.

<!-- - Why is this design the best
- Are there alternative ways to solve this
- Could this be done with existing features or existing solutions -->

# Prior art

-   [MFM (Markup language For Misskey)](https://misskey-hub.net/en/docs/for-users/features/mfm). This is considered to not be a good fit for many reasons:
    -   It is designed for microblogging, not for general-purpose messaging. It is neither consistent with Markdown (in our case RSM) nor is there much use for, for instance, an instant search bar in a messaging app or much of the other fluff that MFM provides.
-   [BBCode](https://en.wikipedia.org/wiki/BBCode). This is considered to not be a good fit for many reasons:
    -   Uses a state machine architecture where Markdown traditionally uses an AST. Additionally it does not fit into the existing Markdown (in our case RSM) syntax and is less ergonomic (requires closing tags, see state machine architecture). It is also less expressive and does not allow for the same extensivility.

<!-- This should include both good and bad outlooks on the proposal. This could include how other platforms, software and hardware solve similar issues if relevent or how any existing proposals have tried to solve the same problem. -->

# Unresolved questions

N/A

# Security concerns

As this RFC does not introduce any new security concerns, there are no security concerns to be addressed.

<!-- How does this RFC impact security - This section might not always be applicable and if you believe it is not, please write your reasoning in this section. -->

# Future ideas

It might be desirable to introduce a permission to Revolt that allows community admins to disable text formatting extensions for specific users or channels.

<!-- Are there any features or changes that this proposal could enable? How does this proposal impact the future of Revolt? -->
