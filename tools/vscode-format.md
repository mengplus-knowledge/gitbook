---
title: 代码格式化对齐功能
slug: vscode-format
categories:
  - 编程设计
tags:
  - vscode
halo:
  site: https://mengplus.top
  name: 3ba74a22-b9cd-4833-b534-538d825aaf1e
  publish: true
cover: ""
---
# 代码格式化对齐功能
如何让代码格式化对齐，让代码看起来更漂亮，不同的工程有不同的格式化要求，比如对齐、缩进等。
每个人独立配置环境难免出现不一致，这里记录一下vscode的格式化功能，使用配置文件，统一格式化。
**使用环境**
 vscode 并安装插件 C/C++
工程路径下放一个`.clang-format`文件
用vscode 打开工程路径， 编写代码，快捷键格式化代码。或代码编辑区右键，选择格式化文档。
```bash
#目录结构
.
├── .clang-format   #就是这个文件
├── .vscode         #vscode配置文件 自动生成不用管
├── Doc
├── Driver
├── Library
├── Project_IAR_7_8
├── README.md
└── Source
```
## 快捷键
ctrl + alt + f 格式化代码
ctrl + shift + alt + f 格式化整个文件
## 配置文件.clang-format
在工程项目路径下 放置一个 `.clang-format`,摘自rtthtread仓库

0. 关闭对齐
    配置文件.clang-format
    ``` yaml
    # Available style options are described in https://clang.llvm.org/docs/ClangFormatStyleOptions.html
    #
    # An easy way to create the .clang-format file is:
    #
    # clang-format -style=llvm -dump-config > .clang-format
    #
    ---
    Language: Cpp
    DisableFormat: true
    ---
    ```
1. vsocode 默认对齐

   不适用任何配置，安装好vscode的插件C/C++ 即可
   ```c
   static const std_gas_conf_t list_std_CH4[] = {
       {0, -0, 0.5, 0.5},
       {0.5, -0.5, 0.5, 0.5},
       {1.5, -0.5, 3.5, 0.003},
       {20, -4, 4, 0.2},
       {35, -4, 4, 0.15},
       {74.81, -5, 5, 0.15},
   };
   
   static void btnm_event_handler(lv_event_t *e)
    {
        lv_event_code_t code = lv_event_get_code(e);
        lv_obj_t *obj = lv_event_get_target(e);
        if (code == LV_EVENT_CLICKED)
        {
            uint32_t id = lv_btnmatrix_get_selected_btn(obj);
            const char *txt = lv_btnmatrix_get_btn_text(obj, id);
            if (txt == NULL)
            {
                return;
            }
        }
    }
   ```

2. 符号对齐

    ```c
    static const std_gas_conf_t list_std_CH4[] = {
        {    0,   -0, 0.5,   0.5},
        {  0.5, -0.5, 0.5,   0.5},
        {  1.5, -0.5, 3.5, 0.003},
        {   20,   -4,   4,   0.2},
        {   35,   -4,   4,  0.15},
        {74.81,   -5,   5,  0.15},
    };
    
    static void btnm_event_handler(lv_event_t *e)
    {
        lv_event_code_t code = lv_event_get_code(e);
        lv_obj_t *obj        = lv_event_get_target(e);
        if (code == LV_EVENT_CLICKED)
        {
            uint32_t id     = lv_btnmatrix_get_selected_btn(obj);
            const char *txt = lv_btnmatrix_get_btn_text(obj, id);
        }
    }
    ```
    配置文件.clang-format
    ``` yaml
    # Available style options are described in https://clang.llvm.org/docs/ClangFormatStyleOptions.html
    #
    # An easy way to create the .clang-format file is:
    #
    # clang-format -style=llvm -dump-config > .clang-format
    #
    ---
    Language: Cpp
    BasedOnStyle: LLVM
    AccessModifierOffset: -1
    AlignAfterOpenBracket: Align
    AlignArrayOfStructures: Right
    AlignConsecutiveAssignments:
    Enabled: true
    AcrossEmptyLines: false
    AcrossComments: false
    AlignCompound: true
    PadOperators: true
    AlignConsecutiveBitFields:
    Enabled: true
    AcrossEmptyLines: false
    AcrossComments: false
    AlignCompound: true
    PadOperators: true
    AlignConsecutiveDeclarations:
    Enabled: false
    AcrossEmptyLines: false
    AcrossComments: false
    AlignCompound: false
    PadOperators: false
    AlignConsecutiveMacros:
    Enabled: true
    AcrossEmptyLines: false
    AcrossComments: false
    AlignCompound: false
    PadOperators: false
    AlignConsecutiveShortCaseStatements:
    Enabled: false
    AcrossEmptyLines: false
    AcrossComments: false
    AlignCaseColons: false
    AlignEscapedNewlines: Left
    AlignOperands: Align
    AlignTrailingComments:
    Kind: Always
    OverEmptyLines: 1
    AllowAllArgumentsOnNextLine: false
    AllowAllParametersOfDeclarationOnNextLine: false
    AllowShortBlocksOnASingleLine: Always
    AllowShortCaseLabelsOnASingleLine: false
    AllowShortEnumsOnASingleLine: false
    AllowShortFunctionsOnASingleLine: None
    AllowShortIfStatementsOnASingleLine: WithoutElse
    AllowShortLambdasOnASingleLine: All
    AllowShortLoopsOnASingleLine: true
    AlwaysBreakAfterDefinitionReturnType: None
    AlwaysBreakAfterReturnType: None
    AlwaysBreakBeforeMultilineStrings: false
    AlwaysBreakTemplateDeclarations: MultiLine
    AttributeMacros:
    - __capability
    BinPackArguments: true
    BinPackParameters: true
    BitFieldColonSpacing: Both
    BraceWrapping:
    AfterCaseLabel: false
    AfterClass: true
    AfterControlStatement: Always
    AfterEnum: true
    AfterExternBlock: false
    AfterFunction: true
    AfterNamespace: true
    AfterObjCDeclaration: true
    AfterStruct: true
    AfterUnion: false
    BeforeCatch: true
    BeforeElse: true
    BeforeLambdaBody: false
    BeforeWhile: false
    IndentBraces: false
    SplitEmptyFunction: true
    SplitEmptyRecord: true
    SplitEmptyNamespace: true
    BreakAfterAttributes: Never
    BreakAfterJavaFieldAnnotations: false
    BreakArrays: false
    BreakBeforeBinaryOperators: NonAssignment
    BreakBeforeConceptDeclarations: Always
    BreakBeforeBraces: Custom
    BreakBeforeInlineASMColon: OnlyMultiline
    BreakBeforeTernaryOperators: true
    BreakConstructorInitializers: AfterColon
    BreakInheritanceList: AfterColon
    BreakStringLiterals: true
    ColumnLimit: 0
    CommentPragmas: "^ IWYU pragma:"
    CompactNamespaces: false
    ConstructorInitializerIndentWidth: 4
    ContinuationIndentWidth: 4
    Cpp11BracedListStyle: true
    DerivePointerAlignment: false
    DisableFormat: false
    EmptyLineAfterAccessModifier: Never
    EmptyLineBeforeAccessModifier: Always
    ExperimentalAutoDetectBinPacking: false
    FixNamespaceComments: true
    ForEachMacros:
    - foreach
    - Q_FOREACH
    - BOOST_FOREACH
    IfMacros:
    - KJ_IF_MAYBE
    IncludeBlocks: Preserve
    IncludeCategories:
    - Regex: '^"(llvm|llvm-c|clang|clang-c)/'
        Priority: 2
        SortPriority: 0
        CaseSensitive: false
    - Regex: '^(<|"(gtest|gmock|isl|json)/)'
        Priority: 3
        SortPriority: 0
        CaseSensitive: false
    - Regex: ".*"
        Priority: 1
        SortPriority: 0
        CaseSensitive: false
    IncludeIsMainRegex: "(Test)?$"
    IncludeIsMainSourceRegex: ""
    IndentAccessModifiers: false
    IndentCaseBlocks: false
    IndentCaseLabels: false
    IndentExternBlock: NoIndent
    IndentGotoLabels: true
    IndentPPDirectives: None
    IndentRequiresClause: true
    IndentWidth: 4
    IndentWrappedFunctionNames: false
    InsertBraces: false
    InsertNewlineAtEOF: true
    InsertTrailingCommas: None
    IntegerLiteralSeparator:
    Binary: 0
    BinaryMinDigits: 0
    Decimal: 0
    DecimalMinDigits: 0
    Hex: 0
    HexMinDigits: 0
    JavaScriptQuotes: Leave
    JavaScriptWrapImports: true
    KeepEmptyLinesAtTheStartOfBlocks: false
    KeepEmptyLinesAtEOF: true
    LambdaBodyIndentation: Signature
    LineEnding: DeriveLF
    MacroBlockBegin: ""
    MacroBlockEnd: ""
    MaxEmptyLinesToKeep: 2
    NamespaceIndentation: None
    ObjCBinPackProtocolList: Auto
    ObjCBlockIndentWidth: 2
    ObjCBreakBeforeNestedBlockParam: true
    ObjCSpaceAfterProperty: false
    ObjCSpaceBeforeProtocolList: true
    PackConstructorInitializers: BinPack
    PenaltyBreakAssignment: 1000
    PenaltyBreakBeforeFirstCallParameter: 19
    PenaltyBreakComment: 300
    PenaltyBreakFirstLessLess: 120
    PenaltyBreakOpenParenthesis: 0
    PenaltyBreakString: 1000
    PenaltyBreakTemplateDeclaration: 10
    PenaltyExcessCharacter: 1000000
    PenaltyIndentedWhitespace: 0
    PenaltyReturnTypeOnItsOwnLine: 1000
    PointerAlignment: Right
    PPIndentWidth: 4
    QualifierAlignment: Leave
    ReferenceAlignment: Pointer
    ReflowComments: false
    RemoveBracesLLVM: false
    RemoveParentheses: Leave
    RemoveSemicolon: false
    RequiresClausePosition: OwnLine
    RequiresExpressionIndentation: OuterScope
    SeparateDefinitionBlocks: Leave
    ShortNamespaceLines: 1
    SortIncludes: Never
    SortJavaStaticImport: Before
    SortUsingDeclarations: LexicographicNumeric
    SpaceAfterCStyleCast: false
    SpaceAfterLogicalNot: false
    SpaceAfterTemplateKeyword: true
    SpaceAroundPointerQualifiers: Both
    SpaceBeforeAssignmentOperators: true
    SpaceBeforeCaseColon: false
    SpaceBeforeCpp11BracedList: false
    SpaceBeforeCtorInitializerColon: true
    SpaceBeforeInheritanceColon: true
    SpaceBeforeJsonColon: false
    SpaceBeforeParens: ControlStatements
    SpaceBeforeParensOptions:
    AfterControlStatements: true
    AfterForeachMacros: true
    AfterFunctionDefinitionName: false
    AfterFunctionDeclarationName: false
    AfterIfMacros: true
    AfterOverloadedOperator: false
    AfterRequiresInClause: false
    AfterRequiresInExpression: false
    BeforeNonEmptyParentheses: false
    SpaceBeforeRangeBasedForLoopColon: true
    SpaceBeforeSquareBrackets: false
    SpaceInEmptyBlock: false
    SpacesBeforeTrailingComments: 1
    SpacesInAngles: Never
    SpacesInContainerLiterals: true
    SpacesInLineCommentPrefix:
    Minimum: 1
    Maximum: -1
    SpacesInParens: Never
    SpacesInParensOptions:
    InCStyleCasts: false
    InConditionalStatements: false
    InEmptyParentheses: false
    Other: false
    SpacesInSquareBrackets: false
    Standard: Latest
    StatementAttributeLikeMacros:
    - Q_EMIT
    StatementMacros:
    - Q_UNUSED
    - QT_REQUIRE_VERSION
    TabWidth: 4
    UseTab: Never
    VerilogBreakBetweenInstancePorts: true
    WhitespaceSensitiveMacros:
    - BOOST_PP_STRINGIZE
    - CF_SWIFT_NAME
    - NS_SWIFT_NAME
    - PP_STRINGIZE
    - STRINGIZE
    ---

## 参考资料
1. [clang-format](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)

