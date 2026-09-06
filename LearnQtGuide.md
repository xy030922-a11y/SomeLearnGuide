# Qt 常用类与 OpenCV 教程目录 / Qt Common Classes and OpenCV Tutorial Contents

> 本教程以 **Qt 6.5.3 + C++17** 为统一基线，依次介绍五个常用模块（`Qt Core`、`Qt GUI`、`Qt Widgets`、`Qt Network`、`Qt SQL`）、`Qt TextToSpeech` 语音合成模块，以及 **OpenCV 4.x C++ API 与 Qt 集成**。OpenCV 是独立的第三方计算机视觉库，需单独配置依赖；示例项目应记录所用 OpenCV 的具体版本。除非特别说明，文中的 Qt API、CMake 配置和示例代码均应兼容 Qt 6.5.3。\
> This tutorial uses **Qt 6.5.3 + C++17** as its consistent baseline and covers five commonly used modules (`Qt Core`, `Qt GUI`, `Qt Widgets`, `Qt Network`, and `Qt SQL`), the `Qt TextToSpeech` speech-synthesis module, and **OpenCV 4.x C++ APIs with Qt integration**. OpenCV is a separate third-party computer vision library that requires its own dependency configuration; example projects should record the exact OpenCV version used. Unless otherwise stated, all Qt APIs, CMake configurations, and examples should be compatible with Qt 6.5.3.

## 教程导读 / Tutorial Guide

1. Qt 简介与版本选择 / Introduction to Qt and version selection
2. Qt Creator、编译器与构建套件 / Qt Creator, compilers, and kits
3. CMake 项目的基本结构、`qt_standard_project_setup()` 与 `qt_add_executable()` / Basic CMake project structure, qt_standard_project_setup(), and qt_add_executable()
4. `moc`、`uic`、`rcc` 与 `AUTOMOC`、`AUTOUIC`、`AUTORCC` / moc, uic, rcc, and CMake automatic code generation
5. Qt Designer、`.ui` 界面文件与 `.qrc` 资源文件 / Qt Designer, .ui forms, and .qrc resource files
6. CMake Presets、Debug/Release 配置与构建目录 / CMake Presets, Debug/Release configurations, and build directories
7. 第一个 Qt 程序 / Your first Qt application
8. Qt 帮助文档的阅读方法 / How to use the Qt documentation
9. Qt 的对象所有权与父子对象机制 / Object ownership and the parent-child mechanism
10. `lupdate`、`lrelease` 与 Qt Linguist 国际化流程 / Internationalization with lupdate, lrelease, and Qt Linguist
11. `windeployqt`、`macdeployqt` 与平台部署工具 / windeployqt, macdeployqt, and platform deployment tools
12. Qt 与 OpenCV 的职责分工、图像处理流程与学习顺序 / Qt and OpenCV responsibilities, image-processing workflows, and learning order

---

## 第一篇：Qt Core——非界面核心类 / Part I: Qt Core — Non-GUI Core Classes

### 第 1 章：QObject 与元对象系统 / Chapter 1: QObject and the Meta-Object System

1. `QObject`：Qt 对象模型的基础 / Foundation of the Qt object model
2. `Q_OBJECT`：启用元对象能力 / Enabling meta-object features
3. `QMetaObject`：运行时类型与方法信息 / Runtime type and method information
4. `QMetaProperty`：属性系统 / The property system
5. `QMetaType`：自定义类型注册 / Custom type registration
6. `QPointer`：安全跟踪 QObject 指针 / Safely tracking QObject pointers
7. `QSharedPointer`、`QWeakPointer` 与 `QScopedPointer` / Shared, weak, and scoped pointers
8. 父子对象与自动内存管理 / Parent-child objects and automatic memory management

### 第 2 章：信号与槽 / Chapter 2: Signals and Slots

1. 信号与槽的基本语法 / Basic signal-slot syntax
2. `QObject::connect()` 的常见写法 / Common QObject::connect() patterns
3. Lambda 表达式作为槽函数 / Using lambda expressions as slots
4. 自动连接与 `QMetaObject::connectSlotsByName()` / Automatic signal-slot connections
5. 直接、队列与阻塞队列连接 / Direct, queued, and blocking queued connections
6. `QObject::sender()` 与信号来源 / Identifying the signal sender
7. `QSignalBlocker`：临时屏蔽信号 / Temporarily blocking signals
8. 自定义信号、槽与参数类型 / Custom signals, slots, and parameter types

### 第 3 章：字符串、字节与通用数据 / Chapter 3: Strings, Bytes, and Generic Data

1. `QString`：Unicode 字符串 / Unicode strings
2. `QStringView`：轻量字符串视图 / Lightweight string views
3. `QByteArray`：二进制字节数组 / Binary byte arrays
4. `QChar`：字符处理 / Character handling
5. `QRegularExpression`：正则表达式 / Regular expressions
6. `QVariant`：通用值容器 / Generic value containers
7. `QAnyStringView`：统一字符串参数 / Unified string parameters
8. 字符串编码与类型转换 / String encoding and type conversion

### 第 4 章：Qt 容器与迭代器 / Chapter 4: Qt Containers and Iterators

1. `QList` 与 `QVector`：顺序容器 / Sequential containers
2. `QMap` 与 `QMultiMap`：有序关联容器 / Ordered associative containers
3. `QHash` 与 `QMultiHash`：哈希容器 / Hash-based containers
4. `QSet`：集合容器 / Set containers
5. `QPair`：二元数据 / Value pairs
6. `QStringList`：字符串列表 / String lists
7. Java 风格与 STL 风格迭代器 / Java-style and STL-style iterators
8. Qt 容器与 STL 容器的转换 / Converting between Qt and STL containers
9. Qt 值语义、复制构造与赋值 / Qt value semantics, copy construction, and assignment
10. 隐式共享、引用计数与浅复制 / Implicit sharing, reference counting, and shallow copies
11. 写时复制、深复制与 `detach()` / Copy-on-write, deep copies, and detach()
12. `const` 正确性与避免意外分离 / Const-correctness and avoiding accidental detachment
13. 隐式共享容器的迭代器失效问题 / Iterator invalidation with implicitly shared containers
14. 隐式共享类的可重入性、线程安全与外部同步 / Reentrancy, thread safety, and external synchronization for implicitly shared classes
15. `QSharedData` 与 `QSharedDataPointer`：自定义隐式共享类 / Building custom implicitly shared classes
16. `QExplicitlySharedDataPointer`：显式共享 / Explicit data sharing
17. 按值传递、按引用传递、移动语义与性能取舍 / Pass-by-value, pass-by-reference, move semantics, and performance tradeoffs
18. `Q_DECLARE_TYPEINFO` 与自定义值类型优化 / Q_DECLARE_TYPEINFO and custom value-type optimization

### 第 5 章：时间、定时器与事件循环 / Chapter 5: Time, Timers, and Event Loops

1. `QDate`、`QTime` 与 `QDateTime` / Dates, times, and date-times
2. `QTimeZone`：时区处理 / Time-zone handling
3. `QElapsedTimer`：耗时测量 / Elapsed-time measurement
4. `QDeadlineTimer`：截止时间控制 / Deadline control
5. `QTimer`：周期任务与延迟执行 / Periodic and delayed execution
6. `QEvent` 与事件类型 / QEvent and event types
7. `QObject::event()` 与事件处理函数 / QObject::event() and event handlers
8. `installEventFilter()` 与 `eventFilter()` / Installing and implementing event filters
9. `QCoreApplication::sendEvent()` 与 `postEvent()` / Sending and posting events
10. 自定义事件、事件接受与传播 / Custom events, event acceptance, and propagation
11. `QEventLoop`：局部事件循环 / Local event loops
12. `QCoreApplication`：应用程序事件循环 / The application event loop

### 第 6 章：文件、目录与数据流 / Chapter 6: Files, Directories, and Data Streams

1. `QFile`：文件读写 / Reading and writing files
2. `QFileInfo`：文件信息 / File information
3. `QDir`：目录操作 / Directory operations
4. `QDirIterator`：目录遍历 / Directory iteration
5. `QIODevice`：输入输出设备抽象 / I/O device abstraction
6. `QTextStream`：文本流 / Text streams
7. `QDataStream`：二进制序列化 / Binary serialization
8. `QSaveFile`：安全写入文件 / Safe file writing
9. `QTemporaryFile` 与 `QTemporaryDir` / Temporary files and directories
10. `QFileSystemWatcher`：监视文件变化 / Monitoring file-system changes
11. `QStandardPaths`：标准系统路径 / Standard system paths
12. `QResource`：Qt 资源系统 / The Qt resource system

### 第 7 章：JSON、XML 与配置 / Chapter 7: JSON, XML, and Settings

1. `QJsonDocument`：JSON 文档 / JSON documents
2. `QJsonObject`：JSON 对象 / JSON objects
3. `QJsonArray`：JSON 数组 / JSON arrays
4. `QJsonValue`：JSON 值 / JSON values
5. `QXmlStreamReader`：流式读取 XML / Streaming XML input
6. `QXmlStreamWriter`：流式写入 XML / Streaming XML output
7. `QSettings`：持久化应用配置 / Persistent application settings
8. 配置文件的版本兼容设计 / Version-compatible configuration design

### 第 8 章：进程、线程与并发 / Chapter 8: Processes, Threads, and Concurrency

1. `QProcess`：启动和管理外部进程 / Starting and managing external processes
2. `QThread`：线程基础 / Thread fundamentals
3. 工作对象移动到线程 / Moving worker objects to threads
4. `QThreadPool` 与 `QRunnable` / Thread pools and runnable tasks
5. `QMutex` 与 `QMutexLocker` / Mutexes and scoped locking
6. `QReadWriteLock`：读写锁 / Read-write locks
7. `QWaitCondition`：线程等待与唤醒 / Thread waiting and wake-up
8. `QSemaphore`：信号量 / Semaphores
9. `QFuture` 与 `QFutureWatcher` / Asynchronous results and monitoring
10. `QPromise`：异步结果、进度与取消 / Asynchronous results, progress, and cancellation
11. `QtConcurrent::run()` 与 `QtConcurrent::task()` / Running functions and building concurrent tasks
12. `mapped()`、`filtered()` 与 Map-Reduce / Concurrent mapping, filtering, and Map-Reduce
13. 异步任务的暂停、取消、进度与异常 / Pausing, canceling, reporting progress, and handling exceptions in asynchronous tasks
14. 跨线程信号槽与线程安全 / Cross-thread signals, slots, and thread safety

### 第 9 章：实用工具类 / Chapter 9: Utility Classes

1. `QPoint` 与 `QPointF`：整数和浮点坐标 / Integer and floating-point coordinates
2. `QSize` 与 `QSizeF`：整数和浮点尺寸 / Integer and floating-point sizes
3. `QRect` 与 `QRectF`：整数和浮点矩形 / Integer and floating-point rectangles
4. `QMargins`：边距表示与运算 / Margin representation and operations
5. `QUrl`：URL 解析与编码 / URL parsing and encoding
6. `QUuid`：唯一标识符 / Universally unique identifiers
7. `QLocale`：本地化格式 / Locale-aware formatting
8. `QTranslator`：界面国际化 / User-interface translation
9. `QCommandLineParser`：命令行参数 / Command-line arguments
10. `QDebug`：调试输出流 / Debug output streams
11. `qDebug()`、`qInfo()`、`qWarning()` 与 `qCritical()` / Debug, informational, warning, and critical messages
12. `QLoggingCategory`：分类日志与日志规则 / Categorized logging and logging rules
13. `Q_ASSERT` 与调试断言 / Q_ASSERT and debug assertions
14. `QRandomGenerator`：随机数 / Random-number generation
15. `QCryptographicHash`：哈希计算 / Cryptographic hashes
16. `QLibrary`：动态库加载 / Dynamic library loading

---

## 第二篇：Qt GUI——图形与窗口基础类 / Part II: Qt GUI — Graphics and Windowing Classes

### 第 10 章：GUI 应用与窗口系统 / Chapter 10: GUI Applications and Windowing

1. `QGuiApplication`：GUI 应用程序入口 / GUI application entry point
2. `QWindow`：原生窗口抽象 / Native window abstraction
3. `QScreen`：屏幕与多显示器 / Screens and multi-monitor environments
4. `QSurface` 与 `QSurfaceFormat` / Rendering surfaces and formats
5. 高 DPI 与设备像素比 / High DPI and device pixel ratios
6. 窗口状态、标志与几何信息 / Window states, flags, and geometry

### 第 11 章：颜色、字体与图像 / Chapter 11: Colors, Fonts, and Images

1. `QColor`：颜色表示与转换 / Color representation and conversion
2. `QPalette`：界面调色板 / User-interface palettes
3. `QFont`：字体设置 / Font configuration
4. `QFontMetrics` 与 `QFontMetricsF` / Font measurements
5. `QImage`：像素级图像处理 / Pixel-level image processing
6. `QPixmap`：面向屏幕显示的图像 / Screen-optimized images
7. `QBitmap`：单色位图 / Monochrome bitmaps
8. `QIcon`：多状态图标 / Multi-state icons
9. `QImageReader` 与 `QImageWriter` / Image reading and writing
10. `QMovie`：动态图像播放 / Animated image playback

### 第 12 章：二维绘图系统 / Chapter 12: The 2D Painting System

1. `QPainter`：绘图入口 / Painting entry point
2. `QPaintDevice`：绘图设备 / Paint devices
3. `QPen`：线条样式 / Line styles
4. `QBrush`：填充样式 / Fill styles
5. `QPainterPath`：复杂路径 / Complex painter paths
6. `QPolygon` 与 `QPolygonF` / Integer and floating-point polygons
7. `QRegion`：区域运算 / Region operations
8. `QTransform`：二维变换 / 2D transformations
9. 渐变、透明度与抗锯齿 / Gradients, opacity, and antialiasing
10. 坐标系统与裁剪区域 / Coordinate systems and clipping

### Qt OpenGL 专题 / Qt OpenGL Topic

1. Qt OpenGL、Qt OpenGL Widgets 模块与 CMake 配置 / Qt OpenGL, Qt OpenGL Widgets, and CMake configuration
2. OpenGL 渲染管线、状态机、颜色缓冲、深度缓冲与模板缓冲 / The OpenGL pipeline, state machine, color, depth, and stencil buffers
3. `QSurfaceFormat`：OpenGL 版本、Profile、缓冲区和多重采样配置 / OpenGL versions, profiles, buffers, and multisampling
4. `QOpenGLContext`：上下文创建、当前上下文与生命周期 / Context creation, current contexts, and lifecycle
5. `QOpenGLFunctions` 与 `QOpenGLExtraFunctions`：跨平台函数访问 / Cross-platform OpenGL function access
6. `QOpenGLVersionFunctionsFactory`：获取特定版本函数集 / Obtaining version-specific OpenGL function sets
7. `QOpenGLWidget` 与 `initializeGL()`、`resizeGL()`、`paintGL()` / QOpenGLWidget and its rendering lifecycle
8. `QOpenGLWindow`：基于窗口的 OpenGL 渲染 / Window-based OpenGL rendering
9. `QOpenGLShader` 与 `QOpenGLShaderProgram`：Shader 编译、链接和 Uniform / Shader compilation, linking, and uniforms
10. `QOpenGLBuffer`：顶点、索引与像素缓冲 / Vertex, index, and pixel buffers
11. `QOpenGLVertexArrayObject`：顶点数组对象 / Vertex array objects
12. `QOpenGLTexture`：纹理创建、采样、过滤和 Mipmap / Texture creation, sampling, filtering, and mipmaps
13. `QOpenGLFramebufferObject`：离屏渲染与后处理 / Offscreen rendering and post-processing
14. `QMatrix4x4`、模型矩阵、观察矩阵与投影矩阵 / Model, view, and projection matrices
15. `QOpenGLPaintDevice`：在 OpenGL 表面使用 `QPainter` / Using QPainter on OpenGL surfaces
16. `QOpenGLDebugLogger`、调试上下文与 OpenGL 错误诊断 / Debug logging, debug contexts, and OpenGL error diagnosis
17. `QOffscreenSurface`：无窗口上下文和后台资源处理 / Offscreen contexts and background resource handling
18. `makeCurrent()`、`doneCurrent()` 与 OpenGL 资源释放 / Current-context management and OpenGL resource cleanup
19. `QOpenGLContext::aboutToBeDestroyed` 与上下文销毁前清理 / Cleanup before context destruction
20. 上下文共享、资源共享与线程规则 / Context sharing, resource sharing, and threading rules
21. 高 DPI、设备像素比、Viewport 与窗口尺寸 / High DPI, device pixel ratios, viewports, and window sizes
22. OpenGL 与 Widgets 组合、透明度、叠放和更新行为 / Combining OpenGL with Widgets, transparency, stacking, and update behavior
23. 三角形、纹理、模型加载、离屏渲染与后处理实践 / Triangle, texture, model loading, offscreen rendering, and post-processing examples
24. OpenGL 版本不匹配、黑屏、上下文丢失和驱动兼容问题 / Version mismatches, blank output, context loss, and driver compatibility issues

### 第 13 章：输入事件 / Chapter 13: Input Events

1. `QInputEvent`：输入事件基类 / Base class for input events
2. `QMouseEvent`：鼠标事件 / Mouse events
3. `QKeyEvent`：键盘事件 / Keyboard events
4. `QWheelEvent`：滚轮事件 / Wheel events
5. `QEnterEvent`：鼠标进入事件 / Enter events
6. `QTouchEvent`：触摸事件 / Touch events
7. `QTabletEvent`：数位板事件 / Tablet events
8. `QInputMethodEvent`：输入法事件 / Input-method events
9. 事件接受、忽略与传播 / Event acceptance, ignoring, and propagation

### 第 14 章：剪贴板与拖放 / Chapter 14: Clipboard and Drag-and-Drop

1. `QClipboard`：系统剪贴板 / The system clipboard
2. `QMimeData`：MIME 数据封装 / MIME data containers
3. `QDrag`：拖放操作 / Drag-and-drop operations
4. `QDragEnterEvent`：拖入事件 / Drag-enter events
5. `QDragMoveEvent`：拖动事件 / Drag-move events
6. `QDropEvent`：放下事件 / Drop events
7. 自定义 MIME 类型 / Custom MIME types

### 第 15 章：GUI 常用辅助类 / Chapter 15: Common GUI Helper Classes

1. `QAction`：用户操作抽象 / User-action abstraction
2. `QActionGroup`：互斥与分组操作 / Exclusive and grouped actions
3. `QShortcut`：快捷键 / Keyboard shortcuts
4. `QKeySequence`：按键序列 / Key sequences
5. `QCursor`：鼠标指针 / Mouse cursors
6. `QValidator`：输入校验基类 / Base class for input validation
7. `QIntValidator`、`QDoubleValidator` 与 `QRegularExpressionValidator` / Numeric and regular-expression validators
8. `QStandardItem` 与 `QStandardItemModel` / Standard data items and models
9. `QDesktopServices`：调用桌面服务 / Accessing desktop services
10. `QUndoCommand` 与 `QUndoStack`：撤销和重做框架 / The undo and redo framework

---

## 第三篇：Qt Widgets——桌面界面控件类 / Part III: Qt Widgets — Desktop UI Classes

### 第 16 章：Widgets 应用基础 / Chapter 16: Widgets Application Basics

1. `QApplication`：Widgets 应用程序入口 / Widgets application entry point
2. `QWidget`：所有控件的基础 / Base class for all widgets
3. `QMainWindow`：主窗口框架 / Main-window framework
4. `QDialog`：对话框基础 / Dialog fundamentals
5. `QFrame`：带边框的基础控件 / Framed base widgets
6. 控件的父子关系与生命周期 / Widget ownership and lifetime
7. 控件的尺寸策略与几何属性 / Size policies and geometry

### 第 17 章：布局管理 / Chapter 17: Layout Management

1. `QLayout`：布局基类 / Base layout class
2. `QHBoxLayout`：水平布局 / Horizontal layouts
3. `QVBoxLayout`：垂直布局 / Vertical layouts
4. `QGridLayout`：网格布局 / Grid layouts
5. `QFormLayout`：表单布局 / Form layouts
6. `QStackedLayout`：堆叠布局 / Stacked layouts
7. `QSpacerItem`：弹性空间 / Flexible spacing
8. `QSizePolicy`：尺寸策略 / Size policies
9. 嵌套布局与自适应窗口 / Nested layouts and responsive windows

### 第 18 章：按钮与选择控件 / Chapter 18: Buttons and Selection Widgets

1. `QAbstractButton`：按钮基类 / Base class for buttons
2. `QPushButton`：命令按钮 / Command buttons
3. `QToolButton`：工具按钮 / Tool buttons
4. `QCheckBox`：复选框 / Check boxes
5. `QRadioButton`：单选按钮 / Radio buttons
6. `QButtonGroup`：按钮分组 / Button groups
7. `QComboBox`：下拉选择框 / Combo boxes
8. `QFontComboBox`：字体选择框 / Font combo boxes

### 第 19 章：文本与数值输入 / Chapter 19: Text and Numeric Input

1. `QLineEdit`：单行文本输入 / Single-line text input
2. `QTextEdit`：富文本编辑 / Rich-text editing
3. `QPlainTextEdit`：纯文本编辑 / Plain-text editing
4. `QTextDocument`：结构化富文本文档 / Structured rich-text documents
5. `QTextCursor` 与 `QTextBlock`：文档遍历和编辑 / Document traversal and editing
6. `QTextFormat` 与 `QTextCharFormat`：文本格式 / Text formatting
7. `QSyntaxHighlighter`：语法高亮 / Syntax highlighting
8. `QAbstractSpinBox`：步进输入基类 / Base class for spin boxes
9. `QSpinBox`：整数输入 / Integer input
10. `QDoubleSpinBox`：浮点数输入 / Floating-point input
11. `QDateEdit`、`QTimeEdit` 与 `QDateTimeEdit` / Date and time input
12. `QKeySequenceEdit`：快捷键输入 / Key-sequence input
13. 输入掩码、校验器与自动补全 / Input masks, validators, and completion
14. `QCompleter`：输入补全 / Input completion

### 第 20 章：信息显示控件 / Chapter 20: Display Widgets

1. `QLabel`：文本与图片显示 / Displaying text and images
2. `QLCDNumber`：数字显示 / LCD-style numeric display
3. `QProgressBar`：进度显示 / Progress display
4. `QSlider`：滑块 / Sliders
5. `QDial`：旋钮 / Dials
6. `QScrollBar`：滚动条 / Scroll bars
7. `QStatusBar`：状态栏 / Status bars
8. `QToolTip`：工具提示 / Tooltips

### 第 21 章：容器与页面控件 / Chapter 21: Container and Page Widgets

1. `QGroupBox`：分组框 / Group boxes
2. `QTabWidget`：标签页 / Tabbed pages
3. `QStackedWidget`：堆叠页面 / Stacked pages
4. `QToolBox`：工具箱 / Toolbox pages
5. `QScrollArea`：滚动区域 / Scroll areas
6. `QSplitter`：可调分隔布局 / Adjustable split layouts
7. `QMdiArea` 与 `QMdiSubWindow` / Multiple-document interfaces
8. `QDockWidget`：停靠窗口 / Dock widgets

### 第 22 章：项目视图与模型/视图 / Chapter 22: Item Views and Model/View Programming

1. `QListWidget` 与 `QListWidgetItem` / List widgets and items
2. `QTreeWidget` 与 `QTreeWidgetItem` / Tree widgets and items
3. `QTableWidget` 与 `QTableWidgetItem` / Table widgets and items
4. `QModelIndex` 与 `QPersistentModelIndex` / Model indexes and persistent model indexes
5. `Qt::ItemDataRole` 与 `Qt::ItemFlags` / Item data roles and item flags
6. `QItemSelectionModel`：选择、当前项与共享选择 / Selection, current items, and shared selections
7. `QAbstractItemView`：视图基类 / Base class for item views
8. `QListView`：列表视图 / List views
9. `QTreeView`：树形视图 / Tree views
10. `QTableView`：表格视图 / Table views
11. `QHeaderView`：表头显示、调整与排序 / Header display, resizing, and sorting
12. `QStringListModel` 与 `QFileSystemModel` / String-list and file-system models
13. `QAbstractItemModel`：模型基类 / Base class for item models
14. `QAbstractListModel` 与 `QAbstractTableModel` / List and table model bases
15. `beginInsertRows()`、`endInsertRows()`、`beginResetModel()` 与 `endResetModel()` / Row insertion notifications and model resets
16. `QSortFilterProxyModel`：排序与过滤 / Sorting and filtering
17. `QStyledItemDelegate`：自定义委托 / Custom item delegates
18. `QDataWidgetMapper`：模型字段与控件映射 / Mapping model fields to widgets
19. 自定义模型、视图与委托 / Building custom models, views, and delegates

### 第 23 章：菜单、工具栏与操作 / Chapter 23: Menus, Toolbars, and Actions

1. `QMenu`：菜单 / Menus
2. `QMenuBar`：菜单栏 / Menu bars
3. `QToolBar`：工具栏 / Toolbars
4. `QAction` 与 `QActionGroup` / Actions and action groups
5. `QWidgetAction`：带控件的操作 / Widget-based actions
6. 上下文菜单 / Context menus
7. 快捷键与状态提示 / Shortcuts and status tips

### 第 24 章：标准对话框 / Chapter 24: Standard Dialogs

1. `QMessageBox`：消息对话框 / Message boxes
2. `QFileDialog`：文件对话框 / File dialogs
3. `QColorDialog`：颜色对话框 / Color dialogs
4. `QFontDialog`：字体对话框 / Font dialogs
5. `QInputDialog`：输入对话框 / Input dialogs
6. `QProgressDialog`：进度对话框 / Progress dialogs
7. 模态与非模态对话框 / Modal and modeless dialogs

### 第 25 章：自定义控件与样式 / Chapter 25: Custom Widgets and Styling

1. 重写 `paintEvent()` 绘制控件 / Painting widgets by overriding paintEvent()
2. 重写鼠标、键盘与尺寸事件 / Overriding mouse, keyboard, and resize events
3. `QStyle` 与 `QProxyStyle` / Styles and proxy styles
4. `QStyleOption` 系列类 / QStyleOption classes
5. Qt 样式表（QSS）/ Qt Style Sheets (QSS)
6. 动态属性与样式刷新 / Dynamic properties and style refresh
7. `QGraphicsEffect`：图形效果 / Graphics effects
8. `QPropertyAnimation`：属性动画 / Property animations
9. 自定义控件的属性、信号和槽 / Properties, signals, and slots in custom widgets

### 第 26 章：Graphics View 框架 / Chapter 26: The Graphics View Framework

1. `QGraphicsScene`：图形场景 / Graphics scenes
2. `QGraphicsView`：场景视图 / Scene views
3. `QGraphicsItem`：图元基类 / Base class for graphics items
4. 常用图元类 / Common graphics item classes
5. 图元选择、碰撞与变换 / Item selection, collision, and transformation
6. 自定义图元 / Custom graphics items

### 第 27 章：桌面集成与打印 / Chapter 27: Desktop Integration and Printing

1. `QSystemTrayIcon`：系统托盘 / System tray icons
2. `QFileIconProvider`：系统文件图标 / System file icons
3. `QPrinter`：打印设备 / Printer devices
4. `QPrintDialog`：打印对话框 / Print dialogs
5. `QPrintPreviewDialog`：打印预览 / Print preview

---

## 第四篇：Qt Network——网络通信类 / Part IV: Qt Network — Networking Classes

### 第 28 章：网络基础 / Chapter 28: Networking Fundamentals

1. Qt 网络模块概览 / Overview of the Qt Network module
2. `QHostAddress`：IP 地址 / IP addresses
3. `QHostInfo`：主机名与 DNS 查询 / Host names and DNS lookups
4. `QNetworkInterface`：本机网络接口 / Local network interfaces
5. `QAbstractSocket`：套接字基类 / Base class for sockets
6. 网络错误处理与超时设计 / Network error handling and timeout design

### 第 29 章：TCP 通信 / Chapter 29: TCP Communication

1. `QTcpSocket`：TCP 客户端 / TCP clients
2. `QTcpServer`：TCP 服务端 / TCP servers
3. 建立连接与断开连接 / Establishing and closing connections
4. 读取、写入与缓冲区 / Reading, writing, and buffering
5. 自定义消息协议与粘包处理 / Custom protocols and message framing
6. 多客户端连接管理 / Managing multiple clients
7. 心跳、重连与超时 / Heartbeats, reconnection, and timeouts

### 第 30 章：UDP 通信 / Chapter 30: UDP Communication

1. `QUdpSocket`：UDP 通信 / UDP communication
2. 数据报的发送与接收 / Sending and receiving datagrams
3. 广播 / Broadcasting
4. 多播 / Multicasting
5. UDP 丢包、乱序与数据边界 / Packet loss, ordering, and datagram boundaries

### 第 31 章：HTTP 客户端 / Chapter 31: HTTP Clients

1. `QNetworkAccessManager`：网络请求管理 / Network request management
2. `QNetworkRequest`：请求配置 / Request configuration
3. `QNetworkReply`：响应处理 / Response handling
4. `QUrlQuery`：查询参数 / Query parameters
5. GET、POST、PUT 与 DELETE 请求 / GET, POST, PUT, and DELETE requests
6. JSON 请求与响应 / JSON requests and responses
7. 表单提交与文件上传 / Form submission and file uploads
8. 文件下载、进度与断点续传 / Downloads, progress, and resuming
9. 重定向、超时与请求取消 / Redirects, timeouts, and cancellation
10. `QNetworkCookie` 与 `QNetworkCookieJar` / Cookies and cookie jars
11. `QAbstractNetworkCache` 与 `QNetworkDiskCache` / Network caching

### 第 32 章：认证、代理与安全连接 / Chapter 32: Authentication, Proxies, and Secure Connections

1. `QAuthenticator`：身份认证 / Authentication
2. `QNetworkProxy`：网络代理 / Network proxies
3. `QSslSocket`：TLS 加密套接字 / TLS-encrypted sockets
4. `QSslConfiguration`：TLS 配置 / TLS configuration
5. `QSslCertificate`：数字证书 / Digital certificates
6. `QSslKey`：加密密钥 / Cryptographic keys
7. `QSslError`：安全连接错误 / Secure-connection errors
8. 证书校验与常见 TLS 错误 / Certificate validation and common TLS errors

### 第 33 章：本地进程间通信 / Chapter 33: Local Inter-Process Communication

1. `QLocalSocket`：本地套接字客户端 / Local socket clients
2. `QLocalServer`：本地套接字服务端 / Local socket servers
3. 单实例应用 / Single-instance applications
4. 本地消息协议设计 / Local message protocol design

---

## 第五篇：Qt SQL——数据库访问类 / Part V: Qt SQL — Database Classes

### 第 34 章：数据库连接 / Chapter 34: Database Connections

1. Qt SQL 模块与驱动概览 / Overview of Qt SQL and database drivers
2. `QSqlDatabase`：创建和管理连接 / Creating and managing connections
3. SQLite 数据库连接 / Connecting to SQLite
4. MySQL 与 PostgreSQL 数据库连接 / Connecting to MySQL and PostgreSQL
5. 多数据库与命名连接 / Multiple databases and named connections
6. 连接生命周期与线程限制 / Connection lifetime and thread restrictions
7. 数据库驱动的部署与排错 / Deploying and troubleshooting database drivers

### 第 35 章：SQL 查询与结果处理 / Chapter 35: SQL Queries and Result Handling

1. `QSqlQuery`：执行 SQL 语句 / Executing SQL statements
2. `QSqlRecord`：读取结果记录 / Reading result records
3. `QSqlField`：字段信息 / Field information
4. `QSqlIndex`：索引信息 / Index information
5. `QSqlError`：错误诊断 / Error diagnostics
6. 参数绑定与预处理语句 / Parameter binding and prepared statements
7. 批量插入与批量更新 / Batch inserts and updates
8. SQL 注入防护 / Preventing SQL injection

### 第 36 章：事务与数据一致性 / Chapter 36: Transactions and Data Consistency

1. 开启、提交与回滚事务 / Starting, committing, and rolling back transactions
2. `QSqlDatabase::transaction()`：开启事务 / Starting a transaction
3. `QSqlDatabase::commit()`：提交事务 / Committing a transaction
4. `QSqlDatabase::rollback()`：回滚事务 / Rolling back a transaction
5. 事务失败处理 / Handling transaction failures
6. 并发访问与锁 / Concurrent access and locking

### 第 37 章：SQL 模型与界面绑定 / Chapter 37: SQL Models and UI Binding

1. `QSqlQueryModel`：只读查询模型 / Read-only query models
2. `QSqlTableModel`：单表编辑模型 / Editable single-table models
3. `QSqlRelationalTableModel`：关系表模型 / Relational table models
4. `QSqlRelationalDelegate`：关系数据委托 / Relational data delegates
5. SQL 模型与 `QTableView` 配合 / Using SQL models with QTableView
6. 编辑策略、提交与撤销 / Edit strategies, submission, and reversion
7. 排序、筛选与分页 / Sorting, filtering, and pagination

### 第 38 章：数据库综合实践 / Chapter 38: Database Practice

1. 数据访问层设计 / Data-access layer design
2. 数据库初始化与版本迁移 / Database initialization and migrations
3. 联系人管理系统 / Contact management system
4. 登录、用户与权限管理 / Login, users, and permissions
5. 日志查询与分页展示 / Log querying and paginated display
6. 数据导入、导出与备份 / Data import, export, and backup

---

## 第六篇：Qt TextToSpeech——文本转语音 / Part VI: Qt TextToSpeech — Speech Synthesis

### 第 39 章：语音合成 / Chapter 39: Speech Synthesis

1. Qt TextToSpeech 模块、后端与 CMake 配置 / The Qt TextToSpeech module, backends, and CMake configuration
2. `QTextToSpeech`：创建语音合成器 / Creating a speech synthesizer
3. `say()`：朗读文本 / Speaking text
4. `QTextToSpeech::State`：状态与错误处理 / State and error handling
5. 音量、语速与音调设置 / Configuring volume, rate, and pitch
6. `QVoice`：枚举并选择语音 / Enumerating and selecting voices
7. 语言与区域设置 / Language and locale selection
8. 暂停、继续与停止朗读 / Pausing, resuming, and stopping speech
9. 信号槽、队列朗读与界面联动 / Signals and slots, speech queues, and UI integration
10. 平台语音引擎的部署与排错 / Deploying and troubleshooting platform speech engines

---

## 第七篇：OpenCV 与 Qt——图像处理与计算机视觉 / Part VII: OpenCV and Qt — Image Processing and Computer Vision

> 本篇采用 OpenCV 4.x 的 C++ 接口，建议先掌握第 8 章的线程基础、第 11 章的图像类以及第 16 章的 Widgets 基础。模块划分与构建选项参考 [OpenCV 官方教程](https://docs.opencv.org/4.x/d9/df8/tutorial_root.html)和[构建配置文档](https://docs.opencv.org/4.x/db/d05/tutorial_config_reference.html)。\
> This part uses the OpenCV 4.x C++ API. Study threading in Chapter 8, image classes in Chapter 11, and Widgets basics in Chapter 16 first. Module coverage and build options follow the [official OpenCV tutorials](https://docs.opencv.org/4.x/d9/df8/tutorial_root.html) and [configuration reference](https://docs.opencv.org/4.x/db/d05/tutorial_config_reference.html).

### 第 40 章：OpenCV 入门与环境配置 / Chapter 40: OpenCV Introduction and Environment Setup

1. OpenCV 简介：图像处理、计算机视觉与 Qt 界面协作 / Introduction to OpenCV, image processing, computer vision, and Qt interfaces
2. `core`、`imgproc`、`imgcodecs`、`videoio` 与可选的 `highgui` 模块 / Core, image processing, image codecs, video I/O, and optional HighGUI modules
3. `features2d`、`calib3d`、`video`、`objdetect` 与 `dnn` 模块概览 / Overview of feature, calibration, video analysis, object detection, and DNN modules
4. 预编译库与源码编译：MSVC、MinGW、目标架构和运行库匹配 / Prebuilt libraries and source builds: matching MSVC, MinGW, target architectures, and runtimes
5. `OpenCV_DIR`、`CMAKE_PREFIX_PATH` 与 `OpenCVConfig.cmake` 的定位 / Locating OpenCVConfig.cmake with OpenCV_DIR and CMAKE_PREFIX_PATH
6. `find_package(OpenCV 4 REQUIRED COMPONENTS core imgproc imgcodecs videoio)` 与按需选择组件 / Finding OpenCV 4 and selecting required components
7. `OpenCV_INCLUDE_DIRS`、`OpenCV_LIBS` 与 `Qt6::Widgets` 的目标级配置 / Target-level configuration of OpenCV include paths, libraries, and Qt6::Widgets
8. Debug/Release 库匹配、动态库搜索路径与常见链接错误 / Matching Debug/Release libraries, runtime library search paths, and common linker errors
9. `opencv_contrib` 与主库版本匹配、`OPENCV_EXTRA_MODULES_PATH` 配置 / Matching opencv_contrib to the main library version and configuring extra modules
10. `CV_VERSION`、`cv::getBuildInformation()` 与第一个 Qt + OpenCV 图像显示程序 / Version and build diagnostics, and a first Qt + OpenCV image viewer
11. `WITH_QT` 是 HighGUI 的 Qt 后端选项；使用 Qt Widgets 显示 OpenCV 图像无需启用该选项 / WITH_QT enables the HighGUI Qt backend and is unnecessary for displaying OpenCV images with Qt Widgets

### 第 41 章：核心数据结构与图像内存 / Chapter 41: Core Data Structures and Image Memory

1. `cv::Mat`：矩阵创建、初始化、`empty()` 与资源管理 / Matrix creation, initialization, empty checks, and resource management
2. `rows`、`cols`、`size()`、`type()`、`depth()` 与 `channels()` / Image dimensions, types, bit depths, and channel counts
3. `CV_8UC1`、`CV_8UC3`、`CV_8UC4`、`CV_16UC1` 与 `CV_32FC1` / Common grayscale, color, high-bit-depth, and floating-point image types
4. `cv::Point`、`cv::Size`、`cv::Rect`、`cv::Scalar` 与 `cv::Vec` / Points, sizes, rectangles, scalar values, and fixed-size vectors
5. `data`、`step`、`elemSize()` 与 `isContinuous()`：行步长和连续内存 / Data pointers, row strides, element sizes, and continuous storage
6. ROI、子矩阵与边界检查 / Regions of interest, submatrices, and bounds checking
7. 浅复制、引用计数与共享数据修改；`clone()`、`copyTo()` 与显式深复制 / Shallow copies, reference counting, shared-data writes, and explicit deep copies
8. 外部缓冲区构造 `cv::Mat` 时的所有权与生命周期 / Ownership and lifetime when wrapping external buffers
9. `at<T>()`、`ptr<T>()` 与逐行像素访问 / Typed element access, row pointers, and pixel traversal
10. `convertTo()`、`split()`、`merge()`、掩码与矩阵运算 / Type conversion, channel operations, masks, and matrix arithmetic

### 第 42 章：图像读写与基础处理 / Chapter 42: Image I/O and Basic Processing

1. `cv::imread()`、`cv::imwrite()` 与读取标志、编码参数 / Image loading, saving, read flags, and encoding parameters
2. `QFile` 配合 `cv::imdecode()`、`cv::imencode()`：中文路径、资源文件与内存图像 / Using QFile with image decoding and encoding for Unicode paths, resources, and memory buffers
3. `cv::cvtColor()`：BGR、RGB、灰度与 HSV 色彩空间 / BGR, RGB, grayscale, and HSV color conversions
4. `cv::resize()`、`cv::flip()`、`cv::rotate()` 与插值方式 / Resizing, flipping, rotation, and interpolation methods
5. `cv::GaussianBlur()`、`cv::medianBlur()` 与 `cv::bilateralFilter()` / Gaussian, median, and bilateral filtering
6. `cv::threshold()`、`cv::adaptiveThreshold()` 与 Otsu 二值化 / Fixed, adaptive, and Otsu thresholding
7. `cv::inRange()`：颜色范围筛选与掩码 / Color-range selection and masks
8. `cv::erode()`、`cv::dilate()` 与 `cv::morphologyEx()` / Erosion, dilation, and morphological operations
9. `cv::Sobel()`、`cv::Laplacian()` 与 `cv::Canny()`：梯度和边缘检测 / Gradients and edge detection
10. `cv::calcHist()`、`cv::equalizeHist()` 与 `cv::createCLAHE()` / Histograms, histogram equalization, and adaptive contrast enhancement
11. `cv::warpAffine()`、`cv::warpPerspective()` 与几何变换 / Affine and perspective transformations
12. `cv::line()`、`cv::rectangle()`、`cv::circle()` 与 `cv::putText()`；使用 `QPainter` 叠加中文标注 / Drawing primitives and text, with QPainter for Chinese annotations
13. 输入尺寸、数据类型、通道数校验与 `cv::Exception` 处理 / Validating dimensions, types, and channels, and handling OpenCV exceptions

### 第 43 章：轮廓、形状与特征匹配 / Chapter 43: Contours, Shapes, and Feature Matching

1. `cv::findContours()` 与 `cv::drawContours()`：轮廓提取和绘制 / Finding and drawing contours
2. 轮廓层级、检索模式与近似方法 / Contour hierarchy, retrieval modes, and approximation methods
3. `cv::contourArea()`、`cv::arcLength()` 与 `cv::moments()` / Contour areas, perimeters, and image moments
4. `cv::boundingRect()`、`cv::minAreaRect()` 与 `cv::minEnclosingCircle()` / Bounding rectangles, rotated rectangles, and enclosing circles
5. `cv::approxPolyDP()`、`cv::convexHull()` 与形状筛选 / Polygon approximation, convex hulls, and shape filtering
6. `cv::connectedComponentsWithStats()`：连通区域标记与统计 / Connected-component labeling and statistics
7. `cv::HoughLinesP()` 与 `cv::HoughCircles()`：直线和圆检测 / Line and circle detection
8. `cv::matchTemplate()` 与 `cv::minMaxLoc()`：模板匹配和结果定位 / Template matching and locating match results
9. `cv::ORB`、`cv::SIFT`、`cv::KeyPoint` 与特征描述子 / ORB, SIFT, keypoints, and feature descriptors
10. `cv::BFMatcher`、`cv::FlannBasedMatcher` 与描述子类型、距离度量匹配 / Feature matching with compatible descriptor types and distance metrics
11. 比率检验、`cv::findHomography()` 与 RANSAC 外点剔除 / Ratio tests, homography estimation, and RANSAC outlier rejection

### 第 44 章：视频文件与摄像头采集 / Chapter 44: Video Files and Camera Capture

1. `cv::VideoCapture`：打开摄像头、视频文件与后端支持的网络流 / Opening cameras, video files, and network streams supported by the backend
2. `isOpened()`、`read()`、`grab()`、`retrieve()` 与 `release()` / Capture state, frame acquisition, frame retrieval, and resource release
3. `CAP_ANY`、`CAP_MSMF`、`CAP_DSHOW` 与 `CAP_FFMPEG`：按平台和构建选择后端 / Selecting capture backends by platform and build configuration
4. 分辨率、帧率、曝光与属性设置结果检查 / Resolution, frame rate, exposure, and checking property-setting results
5. 视频总帧数、当前位置、跳转与时间戳的后端差异 / Backend differences in frame counts, positions, seeking, and timestamps
6. 空帧、文件结束、设备占用、断连与重连处理 / Empty frames, end of file, busy devices, disconnections, and reconnection
7. `cv::VideoWriter`、`fourcc()`、帧尺寸、帧率与编码器兼容 / Video recording, codec identifiers, frame dimensions, frame rates, and encoder compatibility
8. Qt 播放状态、定时调度与后台解码协作 / Coordinating Qt playback state, timing, and background decoding
9. `cv::imshow()`、`cv::waitKey()` 的 HighGUI 调试用途与事件处理；Qt Widgets 中的图像显示流程 / HighGUI debugging and event handling, and image presentation in Qt Widgets

### 第 45 章：cv::Mat 与 Qt 图像集成 / Chapter 45: Integrating cv::Mat with Qt Images

1. `cv::Mat`、`QImage` 与 `QPixmap` 的用途和转换流程 / Roles and conversion workflows for cv::Mat, QImage, and QPixmap
2. `CV_8UC1` 与 `QImage::Format_Grayscale8`：灰度图像转换 / Grayscale image conversion
3. `CV_8UC3` 与 `QImage::Format_BGR888`、`Format_RGB888`：通道顺序和颜色转换 / Channel ordering and conversion for BGR and RGB images
4. BGRA、RGBA、`Format_ARGB32` 与 `Format_RGBA8888`：字节序及预乘 Alpha / Byte order and premultiplied alpha in four-channel images
5. `cv::Mat::step` 与 `QImage::bytesPerLine()`：行对齐、填充和非连续 ROI / Row alignment, padding, and non-contiguous regions of interest
6. 16 位与浮点图像的格式选择、归一化和显示范围 / Format selection, normalization, and display ranges for high-bit-depth and floating-point images
7. 外部缓冲区的有效期、`QImage::copy()` 与 `cv::Mat::clone()` 的拷贝边界 / External-buffer lifetime and explicit copy boundaries
8. `QImage::bits()`、`constBits()`、隐式共享与只读数据访问 / Image buffers, implicit sharing, and read-only data access
9. 在 GUI 线程创建 `QPixmap` 并更新 `QLabel`、自定义 `QWidget` 或 `QGraphicsView` / Creating pixmaps and updating image displays on the GUI thread
10. 等比例缩放、缩放和平移、鼠标坐标映射与交互式 ROI / Aspect-ratio preservation, zooming, panning, coordinate mapping, and interactive regions of interest

> 图像内存与格式转换参考 [OpenCV cv::Mat 文档](https://docs.opencv.org/4.x/d3/d63/classcv_1_1Mat.html)和 [Qt 6.5 QImage 文档](https://doc.qt.io/qt-6.5/qimage.html)。\
> For image memory and format conversion, see the [OpenCV cv::Mat reference](https://docs.opencv.org/4.x/d3/d63/classcv_1_1Mat.html) and [Qt 6.5 QImage reference](https://doc.qt.io/qt-6.5/qimage.html).

### 第 46 章：多线程图像处理与性能 / Chapter 46: Multithreaded Image Processing and Performance

1. 采集、算法处理与界面显示的职责划分 / Separating capture, processing, and display responsibilities
2. `QObject` 工作对象与 `QThread`：在所属线程打开、使用和释放采集设备 / Worker objects and capture-device lifecycle within the owning thread
3. 队列连接、`Q_DECLARE_METATYPE(cv::Mat)` 与 `qRegisterMetaType<cv::Mat>()` / Queued connections and cv::Mat meta-type registration
4. `cv::Mat` 共享缓冲区不提供写时复制：跨线程快照、独占所有权与同步 / Shared Mat buffers without copy-on-write: cross-thread snapshots, exclusive ownership, and synchronization
5. 有界帧队列、丢帧策略与避免信号队列积压 / Bounded frame queues, frame-dropping policies, and preventing queued-signal buildup
6. 停止标志、后端支持的读取超时、线程退出与资源回收 / Stop flags, backend-supported read timeouts, thread shutdown, and resource cleanup
7. `QtConcurrent::run()`、`QFutureWatcher` 与批量图片处理 / Concurrent tasks, future monitoring, and batch image processing
8. 参数防抖、任务版本标记与过期结果丢弃 / Debouncing parameter changes, tagging tasks, and discarding stale results
9. `QElapsedTimer`、`cv::TickMeter`、处理帧率与端到端延迟 / Timing, processing frame rates, and end-to-end latency
10. 缓冲区复用、减少拷贝、ROI 处理与 Qt/OpenCV 线程数量协调 / Buffer reuse, copy reduction, region-based processing, and coordinating thread counts
11. 可选加速：`cv::UMat`、OpenCL 与 `cv::cuda::GpuMat` 的构建依赖和数据传输成本 / Optional acceleration with UMat, OpenCL, and CUDA: build dependencies and transfer costs

### 第 47 章：计算机视觉进阶（可选）/ Chapter 47: Further Computer Vision Topics (Optional)

1. 相机模型、内参、外参与畸变系数 / Camera models, intrinsics, extrinsics, and distortion coefficients
2. `cv::findChessboardCorners()`、`cv::cornerSubPix()` 与 `cv::calibrateCamera()` / Chessboard detection, subpixel refinement, and camera calibration
3. `cv::undistort()`、`cv::initUndistortRectifyMap()` 与 `cv::remap()` / Distortion correction and image remapping
4. `cv::solvePnP()` 与 `cv::projectPoints()`：姿态估计和重投影 / Pose estimation and reprojection
5. `cv::createBackgroundSubtractorMOG2()` 与 `cv::calcOpticalFlowPyrLK()` / Background subtraction and sparse optical flow
6. `cv::CascadeClassifier` 与 `cv::QRCodeDetector`：级联分类器和二维码检测 / Cascade classifiers and QR-code detection
7. `cv::dnn::Net`、`cv::dnn::readNetFromONNX()`、`setInput()` 与 `forward()` / Loading ONNX networks and running inference
8. 模型输入尺寸、归一化、通道顺序、输出解析与 `cv::dnn::NMSBoxes()` / Model preprocessing, output parsing, and non-maximum suppression
9. 模型算子兼容、推理后端、计算目标与可选运行依赖 / Model operator compatibility, inference backends, computation targets, and optional runtime dependencies
10. `cv::FileStorage`：标定参数与算法配置的保存和加载 / Saving and loading calibration parameters and algorithm settings

### 第 48 章：Qt + OpenCV 综合实践与部署 / Chapter 48: Qt + OpenCV Projects and Deployment

1. 图片处理工具：打开、预览、参数调节、效果对比与保存 / An image-processing tool with loading, previews, parameter controls, comparisons, and saving
2. 摄像头预览与录像：设备选择、拍照、录制与状态显示 / Camera preview and recording with device selection, snapshots, and status display
3. 交互式 ROI 与轮廓测量：区域选择、目标计数和结果叠加 / Interactive regions of interest, contour measurements, object counting, and overlays
4. 特征匹配演示：关键点显示、匹配连线与目标定位 / A feature-matching demo with keypoints, match visualization, and object localization
5. 批量图像处理：任务队列、进度、取消与结果导出 / Batch processing with task queues, progress, cancellation, and result export
6. 检测结果与 Qt SQL、Qt Network、Qt TextToSpeech 联动 / Integrating detection results with databases, networking, and speech synthesis
7. Qt 部署工具、OpenCV 动态库、视频后端与模型资源的打包和核验 / Packaging and verifying Qt dependencies, OpenCV libraries, video backends, and model assets
8. 干净环境验证与排错：缺失 DLL、平台插件、编解码器、颜色异常和图像内存失效 / Clean-environment checks for missing libraries, platform plugins, codecs, incorrect colors, and invalid image buffers

---

## 附录 / Appendices

1. Qt 5 到 Qt 6 的常见迁移问题 / Common Qt 5 to Qt 6 migration issues
2. qmake 与 CMake 常用写法对照 / Common qmake and CMake patterns
3. 常用模块的 CMake 链接配置 / CMake linking for common modules
4. Qt 对象所有权与内存排查清单 / Object ownership and memory checklist
5. 信号槽常见错误清单 / Common signal-slot errors
6. Qt 插件与数据库驱动部署 / Deploying Qt plugins and database drivers
7. Windows、Linux 与 macOS 应用发布 / Deploying on Windows, Linux, and macOS
8. 常用类速查表 / Common class quick reference
9. Qt + OpenCV 的 CMake 配置、编译器与运行库匹配清单 / Qt + OpenCV CMake configuration, compiler, and runtime compatibility checklist
10. `cv::Mat` 与 `QImage` 的格式、通道顺序、行步长和生命周期速查 / Quick reference for Mat/QImage formats, channel order, row strides, and lifetimes
11. OpenCV 模块、视频后端与可选加速依赖速查 / Quick reference for OpenCV modules, video backends, and optional acceleration dependencies
