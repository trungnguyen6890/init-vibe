---
name: no-ai-chars
description: "Use when writing, editing, or reviewing any document (Markdown, HTML, text, specs, reports, documentation) to ensure only standard ASCII characters + Vietnamese diacritics are used. Automatically triggers on doc creation/editing. Can also be invoked explicitly to audit and clean an existing file."
---

# No-AI-Chars Skill — Tiêu chuẩn ký tự "human-typed"

## Mục tiêu

Tài liệu do Claude tạo phải dùng **chỉ các ký tự mà người bình thường gõ được trên bàn phím**. Không dùng ký tự đặc biệt mang phong cách AI/typographic mà người dùng thường không gõ.

Rule này áp dụng cho: Markdown, HTML (nội dung + SVG text), tài liệu kỹ thuật, báo cáo, specs, comments trong code khi viết cho con người đọc.

---

## Bảng ký tự CẤM + thay thế chuẩn

### Dấu gạch ngang
| Cấm | Unicode | Thay bằng |
|---|---|---|
| `—` | U+2014 (em dash) | `-` |
| `–` | U+2013 (en dash) | `-` |
| `−` | U+2212 (minus sign) | `-` |

### Dấu ngoặc kép / nháy
| Cấm | Unicode | Thay bằng |
|---|---|---|
| `“` `”` | U+201C U+201D (smart double) | `"` |
| `‘` `’` | U+2018 U+2019 (smart single) | `'` |
| `′` | U+2032 (prime) | `'` |
| HTML: `&ldquo; &rdquo; &lsquo; &rsquo;` | | `"` `'` |
| HTML: `&mdash; &ndash;` | | `-` |

### Mũi tên
| Cấm | Thay bằng |
|---|---|
| `→` (U+2192) | `->` |
| `←` (U+2190) | `<-` |
| `↔` (U+2194) | `<->` |
| `⇔` (U+21D4) | `<=>` |
| `►` `▶` (U+25BA U+25B6) | `>` |
| `◀` (U+25C0) | `<` |
| `▲` (U+25B2) | `^` |
| `▼` (U+25BC) | `v` |

### Dấu chấm / dấu phân cách
| Cấm | Unicode | Thay bằng |
|---|---|---|
| `·` (middle dot) | U+00B7 | `-` hoặc `\|` |
| `…` (ellipsis) | U+2026 | `...` |
| `•` (bullet) | U+2022 | `*` |
| `◦` (white bullet) | U+25E6 | `-` |
| HTML: `&middot; &hellip; &bull;` | | `-` `...` `*` |

### Hình hình học (stars, circles, diamonds, squares)
| Cấm | Ý nghĩa gốc | Thay bằng |
|---|---|---|
| `★` (black star) | milestone/rating | `*` hoặc `[*]` |
| `☆` (white star) | | `*` |
| `●` (filled circle) | primary access | `X` hoặc `[X]` |
| `○` (empty circle) | read-only | `.` hoặc `[ ]` |
| `◐` (half circle) | partial/approve | `D` hoặc `[D]` |
| `■` (black square) | stop/marker | `#` hoặc `[X]` |
| `◆` `◇` (diamond) | decorator | `*` |

### Dấu toán học
| Cấm | Thay bằng |
|---|---|
| `×` (multiplication) | `x` |
| `≥` (greater-equal) | `>=` |
| `≤` (less-equal) | `<=` |
| `≠` (not equal) | `!=` |
| `≈` (approx) | `~=` |

### Check / cross
| Cấm | Thay bằng |
|---|---|
| `✓` `✔` (check) | `v` hoặc `[v]` |
| `✗` `✘` (cross) | `x` hoặc `[x]` |

### Ký tự Hy Lạp trong text thông thường
| Cấm | Thay bằng |
|---|---|
| `σ` (sigma) | `sigma` |
| `α` (alpha) | `alpha` |
| `β` (beta) | `beta` |
| (nếu công thức toán thực sự cần) | giữ LaTeX `$\sigma$` |

### Box drawing (ASCII art)
| Cấm | Thay bằng |
|---|---|
| `─` (light horizontal) | `-` |
| `━` (heavy horizontal) | `=` hoặc `-` |
| `│` (light vertical) | `\|` |
| `┌` `┐` `└` `┘` (corners) | `+` |
| `├` `┤` `┬` `┴` `┼` (intersections) | `+` |

### Warning / emoji
| Cấm | Thay bằng |
|---|---|
| `⚠` (warning) | `[!]` hoặc `WARNING:` |
| Các emoji khác | Từ mô tả rõ ràng |

### HTML numeric entities (decimal > 127)
Cấm dùng icon emoji/special entities trong UI buttons. Thay bằng text rõ ràng:
| Cấm | Thay bằng |
|---|---|
| `&#9733;` (star) | text label không icon |
| `&#128230;` (package emoji) | text label |
| `&#128196;` `&#128200;` `&#9881;` `&#9633;` | text label |
| `&#8226;` (bullet) trong placeholder | `*` |

---

## Ký tự ĐƯỢC PHÉP DÙNG

1. **ASCII in hoa + thường:** `A-Z`, `a-z`
2. **Chữ số:** `0-9`
3. **Dấu câu chuẩn:** `. , ; : ! ? ( ) [ ] { } " ' - + = / \ | * & % $ # @ ~ ^ < > _`
4. **Diacritics tiếng Việt** (đầy đủ): `á à ả ã ạ â ấ ầ ẩ ẫ ậ ă ắ ằ ẳ ẵ ặ đ é è ẻ ẽ ẹ ê ế ề ể ễ ệ í ì ỉ ĩ ị ó ò ỏ õ ọ ô ố ồ ổ ỗ ộ ơ ớ ờ ở ỡ ợ ú ù ủ ũ ụ ư ứ ừ ử ữ ự ý ỳ ỷ ỹ ỵ` (và hoa tương ứng)
5. **HTML safe entities:** `&amp;` `&lt;` `&gt;` `&quot;` `&nbsp;` (khi cần escape)

---

## Cách thực thi

### Khi viết tài liệu mới (auto compliance)
Trước khi tạo/edit file text, Markdown, hoặc HTML với nội dung tiếng Việt/Anh cho người đọc:
- KHÔNG dùng em/en dash - dùng `-`
- KHÔNG dùng smart quotes - dùng `"` `'`
- KHÔNG dùng middle dot - dùng `-` hoặc `\|`
- KHÔNG dùng typographic bullets - dùng `*` hoặc `-`
- KHÔNG dùng arrow symbols trong text - dùng `->` `<-`
- KHÔNG dùng emoji/icon entities trong UI - dùng text labels
- Box drawing chars chỉ dùng nếu thật sự cần cho ASCII diagram - tối ưu là `+ - \|`

### Khi được gọi để clean file có sẵn
Dùng quy trình sau:

1. Scan file tìm tất cả ký tự `ord(ch) > 127` ngoại trừ diacritics tiếng Việt
2. Scan HTML entities `&#N;` với `N > 127` và named entities `&mdash; &ldquo; ...`
3. In bảng thống kê các ký tự tìm được kèm số lượng
4. Áp dụng bảng thay thế ở trên
5. Verify lần cuối - output "CLEAN" khi không còn vi phạm
6. Kiểm tra structural integrity (balanced HTML tags, Markdown heading levels)

### Script mẫu Python

```python
import re
from collections import Counter

VN = set("áàảãạâấầẩẫậăắằẳẵặđéèẻẽẹêếềểễệíìỉĩịóòỏõọôốồổỗộơớờởỡợúùủũụưứừửữựýỳỷỹỵ"
         "ÁÀẢÃẠÂẤẦẨẪẬĂẮẰẲẴẶĐÉÈẺẼẸÊẾỀỂỄỆÍÌỈĨỊÓÒỎÕỌÔỐỒỔỖỘƠỚỜỞỠỢÚÙỦŨỤƯỨỪỬỮỰÝỲỶỸỴ")

REPLACEMENTS = {
    # Dashes
    '—': '-', '–': '-', '−': '-',
    # Quotes
    '‘': "'", '’': "'", '“': '"', '”': '"', '′': "'",
    # Arrows
    '→': '->', '←': '<-', '↔': '<->', '⇔': '<=>',
    '►': '>', '▶': '>', '◀': '<',
    '▲': '^', '▼': 'v',
    # Dots/bullets
    '·': '-', '…': '...', '•': '*', '◦': '-',
    # Shapes
    '★': '*', '☆': '*',
    '●': 'X', '○': '.', '◐': 'D',
    '■': '#', '□': '[ ]', '◆': '*', '◇': '*',
    # Math
    '×': 'x', '≥': '>=', '≤': '<=',
    '≠': '!=', '≈': '~=',
    # Checks
    '✓': 'v', '✗': 'x', '✔': 'v', '✕': 'x',
    # Greek letters (in text, not formulas)
    'σ': 'sigma', 'α': 'alpha', 'β': 'beta',
    # Box drawing
    '─': '-', '━': '=', '│': '|', '┃': '|',
    '┌': '+', '┐': '+', '└': '+', '┘': '+',
    '├': '+', '┤': '+', '┬': '+', '┴': '+', '┼': '+',
    # Warning
    '⚠': '[!]',
    # Misc
    ' ': ' ', '⌐': '',
}

HTML_REPLACEMENTS = {
    '&mdash;': '-', '&ndash;': '-',
    '&ldquo;': '"', '&rdquo;': '"', '&lsquo;': "'", '&rsquo;': "'",
    '&hellip;': '...', '&middot;': '-', '&bull;': '*', '&times;': 'x',
    '&#8226;': '*', '&#9733;': '*',
    # Remove decorative icon entities
    '&#128230;': '', '&#128196;': '', '&#128200;': '', '&#9881;': '',
    '&#9633;': '', '&#128101;': '', '&#9876;': '', '&#128250;': '',
    '&#128202;': '', '&#128172;': '', '&#9997;': '', '&#128269;': '',
    '&#129302;': '', '&#127760;': '', '&#127942;': '', '&#128274;': '',
}

def scan_file(path):
    with open(path) as f: content = f.read()
    special = Counter()
    for ch in content:
        if ord(ch) > 127 and ch not in VN and ch != '\n':
            special[ch] += 1
    ents = Counter(re.findall(r'&#\d+;', content))
    ents.update(re.findall(r'&(?:mdash|ndash|ldquo|rdquo|lsquo|rsquo|hellip|middot|bull|times);', content))
    return special, ents

def clean_file(path):
    with open(path) as f: content = f.read()
    for src, dst in REPLACEMENTS.items(): content = content.replace(src, dst)
    for src, dst in HTML_REPLACEMENTS.items(): content = content.replace(src, dst)
    with open(path, 'w') as f: f.write(content)
```

---

## Ngoại lệ (context-dependent)

- **Code blocks** trong docs (inline code hoặc fenced blocks) - KHÔNG thay thế, giữ nguyên syntax ngôn ngữ gốc.
- **Formulas toán học** trong LaTeX hoặc MathJax - giữ nguyên Greek letters, toán symbols.
- **URL / Link** - giữ nguyên đúng địa chỉ.
- **File path / filename** có ký tự đặc biệt - giữ nguyên theo đúng tên thực.
- **Emoji trong test fixture / seed data** cho ứng dụng emoji-aware - giữ nguyên theo yêu cầu nghiệp vụ.

Khi có ngoại lệ hợp lý, nêu rõ với người dùng để xác nhận.

---

## Workflow triệu gọi skill

### Kịch bản 1 - Đang viết doc mới
Tự động tuân thủ. Không cần trigger.

### Kịch bản 2 - Clean file có sẵn
User: "clean file X", "bỏ ký tự đặc biệt trong file Y", hoặc `/no-ai-chars FILE`

Thực hiện:
1. Đọc file X (nhiều file cùng lúc OK)
2. Scan và in bảng thống kê
3. Áp dụng replacements
4. Re-scan verify CLEAN
5. Báo cáo delta (bao nhiêu ký tự đổi, size delta)

### Kịch bản 3 - Review PR/diff
Khi review diff của tài liệu, kiểm tra xem có ký tự mới vi phạm không. Nếu có, đề xuất replacement.

---

## Lý do tại sao rule này quan trọng

1. **Dễ gõ lại/sửa:** người dùng dùng bàn phím chuẩn không gõ được `—` `"` `•` - họ sẽ copy-paste hoặc dùng bộ gõ đặc biệt, gây friction.
2. **Consistency:** Mix `-` và `—` trong cùng document trông thiếu chuyên nghiệp.
3. **Cross-system safe:** ASCII + Vietnamese diacritics hiển thị đúng mọi nơi (terminal, editor, database, email client cũ). Ký tự Unicode đặc biệt có thể bị replace thành `?` hoặc mojibake.
4. **Chuẩn tài liệu nhà nước:** Văn bản hành chính VN không dùng em dash, smart quote, emoji - chỉ dùng ký tự ASCII + tiếng Việt chuẩn.
5. **Tránh AI-signature:** Em dash, smart quote là "dấu vân tay" của LLM. Văn bản cần phong cách con người gõ thật.

---

## Checklist trước khi hoàn thành tài liệu

- [ ] File đã scan, không còn ký tự `ord > 127` ngoài diacritics tiếng Việt
- [ ] Không còn HTML entity `&#N;` với N > 127 (trừ khi user yêu cầu)
- [ ] Không còn `&mdash; &ndash; &ldquo; &rdquo;` trong HTML
- [ ] Cấu trúc file còn nguyên vẹn (balanced tags, headings hierarchy)
- [ ] Smart quotes auto-insert đã được tắt nếu dùng editor có autocorrect
