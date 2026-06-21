# `0x0000` Bamboo Code Specification (BBC)

[简体中文](README.zh-Hans.md)

Bamboo Code (BBC) is a Markdown-only tree-structured coding specification.
It assigns stable hexadecimal codes to Markdown directories, Markdown files, and headings.
Those codes describe the position of each node in a documentation tree.

BBC is intentionally small.
It uses coding rules to keep Markdown documentation trees clear, contiguous, and easy to validate.
Every code maps to one fixed-width layout, and every hexadecimal digit in that layout has one structural role.

---

## `0x1000` Basic Model

### `0x1100` Goal

BBC uses short codes to express Markdown documentation nodes in a tree.
The same node should have the same code in source, rendered Markdown, and validators.
Coding rules describe structure only, not content meaning.

### `0x1200` Nodes

BBC encodes three kinds of nodes.

| Node | Meaning |
| :--- | :--- |
| Directory | A directory node in the Markdown documentation tree. |
| Independent file | A coded Markdown file other than a `README.md` carrier. |
| Heading | A heading from H1 through H5 inside a Markdown file. |

Every directory has a carrier context.
If the directory contains `README.md`, that file carries the directory context.
`README.md` is not an independent file and does not consume a file ordinal.

### `0x1300` Full Layout

Every code first recovers an 8-digit full layout.
BBC always encodes from high digits to low digits.
Parent structure occupies higher digits before child structure occupies lower digits.

```text
0x D1 D2 D3  F-H1 H2 H3 H4 H5
   └───────────╂────────────┘
    Directory  ┃    Heading
           File & H1
```

`D1 D2 D3` express the directory path.
When the directory depth is less than 3, directory digits are filled from the left.
`D1` is the highest-level directory.
`D2` is a child directory of `D1`.
`D3` is a child directory of `D2`.

`F-H1` selects the carrier or an independent file in the current directory.
`F-H1=0` means the directory carrier.
`F-H1` values from `1` through `F` mean independent files.
The same `F-H1` is also the H1 digit.

`H2 H3 H4 H5` express the heading path below H1.
BBC does not encode H6.
BBC-governed Markdown does not use H6.

### `0x1400` Capacity

`1` through `F` are usable child ordinals.
`0` is not an ordinal.
`0` means an empty structural slot.
In the `F-H1` digit, `0` specifically means the directory carrier.

BBC encodes at most 3 directory levels.
One parent can encode at most 15 children at one level.
Each directory can have one `README.md` carrier and 15 independent files.
Each carrier or independent file can encode H1 through H5.

### `0x1500` Terms

| Term | Definition |
| :--- | :--- |
| Bamboo Code | A `0x`-prefixed hexadecimal code assigned to a documentation node. |
| Full layout | The 8-digit layout `0xD1D2D3FH2H3H4H5`, where `F` is the `F-H1` digit. |
| Heading code | A Bamboo Code written in a Markdown heading. |
| Name code | The `0xD1D2D3F` code written in a directory name or independent file name, where `F` is the `F-H1` digit. |
| Directory digits | `D1 D2 D3`. |
| File-H1 digit | `F-H1`. |
| Heading digits | `F-H1 H2 H3 H4 H5`. |
| Directory carrier | A `README.md` that represents its directory context. |
| Independent file | A coded Markdown file other than a directory carrier. |

---

## `0x2000` Written Rules

### `0x2100` Heading Codes

Markdown headings must write heading codes.
A heading code is `0x` followed by 1 to 8 uppercase hexadecimal digits.
The code in a heading must be wrapped in backticks.

```markdown
## `0x2000` Written Rules
```

When parsing a heading code, first validate omission width under `0x2500`, then left-pad to the 8-digit full layout.
A heading code can represent H1 through H5.

### `0x2200` Name Codes

Directory names and independent file names must write name codes.
A name code is `0x` followed by 1 to 4 uppercase hexadecimal digits.
A name code contains only `D1D2D3F`, where `F` is the `F-H1` digit.
`H2H3H4H5` is fixed as `0000` and is not written into file-system names.

When parsing a name code, first validate omission width under `0x2500`.
Then left-pad to 4 digits as `D1D2D3F`, and append `0000` on the right.
A name code can only represent a directory carrier or an independent-file H1.
Human-readable text may follow the code.

```text
0x1000-docs/
0x1001-guide.md
```

### `0x2300` Carrier File Names

The carrier file name is fixed as `README.md`.
`README.md` itself does not write a name code.
It uses the carrier code of its containing directory.

The documentation root is also a directory context.
The root carrier full layout is `0x00000000`.
`README.md` at the root uses the root carrier code.

### `0x2400` Written Positions

| Position | Written form | Recovery |
| :--- | :--- | :--- |
| Markdown heading | Heading code. | Left-pad to 8 digits. |
| Directory name | Name code. | Left-pad to 4 digits, then append `0000`. |
| Independent file name | Name code. | Left-pad to 4 digits, then append `0000`. |
| Carrier file name | `README.md`. | Use the carrier code of the containing directory. |

The same node may have different written forms in different positions.
Those forms must recover the same full layout.
The same written code may appear in different positions.
Recover it according to its written position.

### `0x2500` Leading-Zero Omission

Leading zeros may only be omitted uniformly within a governed scope.
A directory name code and the heading codes in its README belong to the same scope.
An independent file name code and the heading codes in its body belong to the same scope.
The root README has no name code, so its scope contains only its own heading codes.

Omission width is the number of contiguous leading `0` digits removed from the left side of the canonical digit string.
The canonical digit string for a name code is the 4-digit `D1D2D3F`, where `F` is the `F-H1` digit.
The canonical digit string for a heading code is the 8-digit full layout.
All codes in one governed scope must use the same omission width.
After that shared number of leading `0` digits is removed, every remaining digit must be preserved unchanged.

For example, the canonical digit string of an independent root-file name code is `0001`.
The corresponding H1 heading-code canonical digit string is `00010000`.
If the scope uses omission width 3, they are written as `0x1` and `0x10000`.

As another example, the canonical digit string of the first one-level directory name code is `1000`.
It has no leading `0`, so its omission width can only be 0.
It must be written as `0x1000`, not `0x1`.

If any code in the scope lacks enough leading `0` digits, that omission width is invalid.
Omission changes only written length, not code meaning.

---

## `0x3000` Coding Rules

### `0x3100` Directory Paths

The directory path is written into `D1 D2 D3`.
The path is numbered from the documentation root downward and is left-aligned in the layout.
Unused lower directory digits remain `0`.

| Directory depth | Directory-digit shape | Meaning |
| :--- | :--- | :--- |
| 0 | `000` | Documentation root. |
| 1 | `a00` | The `a`th directory under the root. |
| 2 | `ab0` | The `b`th directory under the `a00` directory. |
| 3 | `abc` | The `c`th directory under the `ab0` directory. |

`a`, `b`, and `c` are ordinals from `1` through `F`.
Directory digits cannot skip a level.

### `0x3200` Carriers and Independent Files

The `F-H1` digit selects a carrier or an independent file in the current directory context.
`F-H1=0` means the directory carrier.
`F-H1` values from `1` through `F` mean independent files in that directory.

The directory carrier does not participate in independent-file ordering.
The first independent file always uses `F-H1=1`.

| `D1D2D3F` | Shortest name code | Meaning |
| :--- | :--- | :--- |
| `0000` | Not used in names | Root directory carrier. |
| `0001` | `0x1` | First independent file under the root. |
| `0002` | `0x2` | Second independent file under the root. |
| `1000` | `0x1000` | Carrier of the first child directory under the root. |
| `1001` | `0x1001` | First independent file under that child directory. |
| `1100` | `0x1100` | Carrier of the first two-level directory path. |
| `1101` | `0x1101` | First independent file under that two-level path. |
| `1110` | `0x1110` | Carrier of the first three-level directory path. |
| `1111` | `0x1111` | First independent file under that three-level path. |

The table uses shortest forms for explanation.
Actual file-system names must still follow the shared omission width in `0x2500`.

### `0x3300` Heading Paths

BBC encodes H1 through H5.
H1 uses the full layout of its carrier or independent file.
H2 through H5 use `H2 H3 H4 H5` in order.

| Markdown level | Code rule |
| :--- | :--- |
| H1 in a carrier | Uses the carrier code with `F-H1=0`. |
| H1 in an independent file | Uses the independent-file code with non-zero `F-H1`. |
| H2 | Sets `H2`. |
| H3 | Sets `H3` under a non-zero `H2`. |
| H4 | Sets `H4` under non-zero `H2 H3`. |
| H5 | Sets `H5` under non-zero `H2 H3 H4`. |

Root README heading examples:

| Markdown heading | Full layout |
| :--- | :--- |
| H1 | `0x00000000` |
| First H2 | `0x00001000` |
| First H3 under the first H2 | `0x00001100` |
| First H4 under that H3 | `0x00001110` |
| First H5 under that H4 | `0x00001111` |
| Second H2 | `0x00002000` |

Independent root-file heading examples:

| Markdown heading | Full layout |
| :--- | :--- |
| H1 | `0x00010000` |
| First H2 | `0x00011000` |
| First H3 under the first H2 | `0x00011100` |
| Second H2 | `0x00012000` |

### `0x3400` Child Ordinals

Coded children under the same parent and at the same level use `1` through `F`.
The first child uses `1`.
The second child uses `2`.
Later children increase from there.

This rule applies to directory children, independent-file children, and heading children.
A directory carrier is not an independent-file child.

### `0x3500` Continuity

Directory digits must be a contiguous left-aligned prefix.
The only valid shapes are `000`, `a00`, `ab0`, and `abc`.

Heading digits must be a contiguous left-aligned prefix.
If a parent heading digit is `0`, every child heading digit after it must also be `0`.

```text
0x10100000  # invalid: the directory path skips the middle level.
0x00000100  # invalid: H3 cannot exist without H2.
0x00001010  # invalid: H4 cannot exist without H3.
```

### `0x3600` Dense Ordering

Children under the same parent and at the same level must be dense and increasing.
Always use the next available hexadecimal digit.
Do not reserve gaps for future insertion.

```text
One-level directory prefixes:  0x10000000, 0x20000000, 0x30000000, ...
Root independent files:        0x00010000, 0x00020000, 0x00030000, ...
Independent files under D1=1:  0x10010000, 0x10020000, 0x10030000, ...
Root README H2:                0x00001000, 0x00002000, 0x00003000, ...
README H2 under D1=1:          0x10001000, 0x10002000, 0x10003000, ...
H2 in the D1=1 F-H1=1 file:    0x10011000, 0x10012000, 0x10013000, ...

0x00003000  # invalid if 0x00002000 does not exist.
```

---

## `0x4000` Progressive Examples

### `0x4100` Root README

```text
Code        Markdown tree

0x0000  README.md
        └── # `0x0000` Guide
0x1000      └── ## `0x1000` Install
0x1100          └── ### `0x1100` macOS
0x1110              └── #### `0x1110` Homebrew
0x1111                  └── ##### `0x1111` Prefix
0x2000      └── ## `0x2000` Reference
0x2100          └── ### `0x2100` Layout
0x2110              └── #### `0x2110` Digits
0x2111                  └── ##### `0x2111` Directory Digits
```

### `0x4200` Independent Root Files

The root README is a directory carrier.
Independent root files start at `F-H1=1`.

```text
Code        Markdown tree

0x0000      README.md
            └── # `0x0000` Project

0x10000     0x1-overview.md
            └── # `0x10000` Overview
0x11000         └── ## `0x11000` Quick Start

0x20000     0x2-reference.md
            └── # `0x20000` Reference
```

### `0x4300` One-Level Directory

The directory code uses `F-H1=0`.
The directory README uses the same carrier code.
Independent files under that directory start at `F-H1=1`.

```text
Code        Markdown tree

0x10000000  0x1000-docs/
            └── README.md
                └── # `0x10000000` Docs
0x10001000          └── ## `0x10001000` Overview
0x10001100              └── ### `0x10001100` Scope

0x10010000      └── 0x1001-guide.md
                    └── # `0x10010000` Guide
0x10011000          └── ## `0x10011000` Install
0x10011100              └── ### `0x10011100` macOS

0x10020000      └── 0x1002-reference.md
                    └── # `0x10020000` Reference
```

### `0x4400` Three-Level Directory

```text
Code        Markdown tree

0x10000000  0x1000-docs/
0x11000000      └── 0x1100-api/
0x11100000          └── 0x1110-v1/
                        └── README.md
                            └── # `0x11100000` v1
0x11101000                    └── ## `0x11101000` Overview

0x11110000              └── 0x1111-endpoints.md
                            └── # `0x11110000` Endpoints
0x11111000                    └── ## `0x11111000` List
0x11111100                        └── ### `0x11111100` Request
0x11111110                            └── #### `0x11111110` Headers
0x11111111                                └── ##### `0x11111111` Authorization

0x11120000              └── 0x1112-schemas.md
                            └── # `0x11120000` Schemas
```

---

## `0x5000` Grammar and Parsing

### `0x5100` EBNF

```ebnf
hex_upper    = "A" | "B" | "C" | "D" | "E" | "F" ;
hex          = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" | hex_upper ;
hex_1_4      = hex
             | hex , hex
             | hex , hex , hex
             | hex , hex , hex , hex ;
hex_1_8      = hex_1_4
             | hex , hex , hex , hex , hex
             | hex , hex , hex , hex , hex , hex
             | hex , hex , hex , hex , hex , hex , hex
             | hex , hex , hex , hex , hex , hex , hex , hex ;

heading_code = "0x" , hex_1_8 ;
name_code    = "0x" , hex_1_4 ;
```

### `0x5200` Parsing Steps

A parser handles codes in four steps.

1. Identify a heading code, name code, or carrier file name by written position.
2. Validate omission width in the governed scope under `0x2500`.
3. Recover the full layout under `0x2400`.
4. Validate directory, file, heading, and ordering structure under `0x3000`.

Padding only restores omitted empty slots.
Padding does not create nodes.

### `0x5300` Semantic Validation

Semantic validation reads the full layout from left to right.
`D1 D2 D3` select the directory path.
Non-zero directory digits are directory child ordinals.
Directory digits must be contiguous, left-aligned, and dense.

`F-H1=0` selects the carrier context for that directory.
`F-H1` values from `1` through `F` select independent files under that directory.
Non-zero `F-H1` digits must be contiguous and dense.
The same `F-H1` is also the H1 code for the selected carrier or independent file.

`H2 H3 H4 H5` select the heading path below H1.
Non-zero heading digits must be contiguous, left-aligned, and dense.
Every existing child always uses an ordinal from `1` through `F`.
