# Clangd 使用笔记

## 配置

### 配置文件

```yaml title="~/.config/clangd/config.yaml"
If:                               # Apply this config conditionally
  PathMatch: [.*\.h, .*\.cpp, .*\.cpp, .*\.cc]  # to all headers...
  PathExclude: /home/zj/repos/grpc/.*
CompileFlags:                        # Tweak the parse settings
  Add: [-std=c++20, -I/usr/include/python3.10, -x, c++]  # treat all files as C++, enable more warnings
Diagnostics:
  ClangTidy:
    FastCheckFilter: Strict
    Add:
      - bugprone-*
      - modernize-*
      - performance-*
      - cpp-core-guidelines-*
      - clang-analyzer-*
      - cert-*
      - concurrency-*
      - google-*
      - readability-*
    Remove:
      - bugprone-unchecked-optional-access
      - bugprone-easily-swappable-parameters
      - bugprone-lambda-function-name
      - modernize-use-trailing-return-type
      - modernize-use-nodiscard
      - modernize-use-using
      - readability-identifier-length
      - readability-magic-numbers
      - readability-function-cognitive-complexity
      - readability-redundant-access-specifiers
      - readability-implicit-bool-conversion
      - readability-braces-around-statements
      - google-readability-braces-around-statements
  UnusedIncludes: Strict

---

If:
  PathMatch:
    - /usr/include/.*
Diagnostics:
  Suppress: "*"
```
