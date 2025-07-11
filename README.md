<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تنفيذ الاستراتيجية - ديوان الوقف الشيعي</title>
    
    <!-- Chosen Palette: Calm Harmony Neutrals -->
    <!-- Application Structure Plan: A dashboard-centric single-page application. A persistent sidebar on the right provides primary navigation between the main modules (Dashboard, HR, Operations, Organizational Structure, Directorates, Reports). The main content area dynamically displays the selected module. This structure offers a familiar and efficient user experience for a management system, allowing quick access to different functionalities while maintaining context. HR, Operations, and Reports modules now feature full CRUD (Create, Read, Update, Delete) functionality for their respective data. Organizational structure nodes are now clickable to reveal details in a modal. -->
    <!-- Visualization & Content Choices: 
        - Goal: Inform (High-Level KPIs) -> Presentation: Cards with large numbers and donut charts (Chart.js) on the main dashboard for the 4 BSC perspectives. Interaction: Tooltips on hover. Justification: Provides a quick, scannable overview of organizational health.
        - Goal: Organize (Employee Data) -> Presentation: Interactive HTML table with search, add, edit, and delete functionality (HTML/JS). Interaction: Clicking a row reveals detailed employee profile in a dedicated view, and edit/delete buttons for direct manipulation. Justification: Efficiently manages and displays lists of personnel with full data lifecycle control.
        - Goal: Organize (Correspondence, Announcements, Activities, Visit Reports) -> Presentation: Interactive HTML tables for listing items, with dedicated modal forms for adding new entries. Detail views for each item. Interaction: Buttons for "Add", "View", "Edit", "Delete" (where applicable), form submission for data capture. Justification: Provides a structured way to manage and track various operational and reporting elements, allowing for dynamic data entry and review.
        - Goal: Compare (Performance Trends) -> Presentation: Line chart (Chart.js) for financial data over time (placeholder). Interaction: Tooltips and potential filtering. Justification: Clearly visualizes trends and changes.
        - Goal: Inform (Detailed Data) -> Presentation: Tabbed interface for employee profiles (Personal, CV, Leave, etc.). Justification: Organizes complex information into digestible chunks.
        - Goal: Organize (Daily Tasks) -> Presentation: A simple timeline structure (HTML/CSS) for the daily activity log. Justification: Provides a clear, chronological view of tasks.
        - Goal: Organize (Organizational Structure) -> Presentation: Nested HTML elements with Tailwind CSS to form a hierarchical diagram. Clicking nodes opens a generic detail modal with specific unit information. Justification: Clearly visualizes the reporting lines and departments and adds interactivity for detail viewing.
        - Goal: Analyze (Reports) -> Presentation: Tables and descriptive text sections to outline comprehensive performance analysis, capabilities, obstacles, and needs. Visit Reports now include a 'Visit Committee' field. This section now has CRUD operations. Justification: Provides structured information and prompts for deeper analysis without requiring complex backend data processing within the frontend.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->

    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Cairo', sans-serif;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        .sidebar-icon {
            font-size: 1.5rem;
            line-height: 1;
        }

        /* Organizational Chart Styling */
        .org-chart .node {
            background-color: #f0f9ff; /* sky-50 */
            border: 1px solid #bae6fd; /* sky-200 */
            border-radius: 0.5rem;
            padding: 1rem;
            margin: 0.5rem;
            text-align: center;
            box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
            font-size: 0.9rem;
            font-weight: 600;
            color: #0369a1; /* sky-700 */
            position: relative;
            cursor: pointer; /* Make nodes clickable */
            transition: background-color 0.2s ease-in-out;
        }
        .org-chart .node:hover {
            background-color: #e0f2fe; /* sky-100 */
        }
        .org-chart .connector {
            position: absolute;
            background-color: #94a3b8; /* slate-400 */
        }
        .org-chart .vertical-line {
            width: 2px;
            height: 1.5rem;
            left: 50%;
            transform: translateX(-50%);
            top: 100%;
        }
        .org-chart .horizontal-line {
            height: 2px;
            width: calc(100% - 1rem); /* adjust based on node margin/padding */
            top: 0; /* Align with top of current row of children */
            left: 0.5rem;
        }
        .org-chart .children {
            display: flex;
            justify-content: center;
            margin-top: 1rem;
            position: relative;
        }
        .org-chart .children::before {
            content: '';
            position: absolute;
            width: 2px;
            height: 1rem;
            background-color: #94a3b8; /* slate-400 */
            top: -1rem;
            left: 50%;
            transform: translateX(-50%);
        }
        .org-chart .children > div {
            position: relative;
            margin: 0 0.5rem; /* Space between children nodes */
        }
        .org-chart .children > div::before {
            content: '';
            position: absolute;
            width: 2px;
            height: 1rem;
            background-color: #94a3b8; /* slate-400 */
            top: -1rem;
            left: 50%;
            transform: translateX(-50%);
        }
        /* Connect horizontal lines */
        .org-chart .children:not(.flex-col)::after {
            content: '';
            position: absolute;
            height: 2px;
            background-color: #94a3b8; /* slate-400 */
            top: -1rem; /* Align with vertical lines connecting to parent */
            left: 0;
            right: 0;
            transform: translateY(-50%);
        }
        /* Adjust horizontal line for first/last child */
        .org-chart .children > div:first-child::before {
             left: 50%; /* Vertical line for first child */
             transform: translateX(-50%);
        }
        .org-chart .children > div:last-child::before {
            left: 50%; /* Vertical line for last child */
            transform: translateX(-50%);
        }
         .org-chart .children.single-child::after {
            display: none; /* No horizontal line for a single child */
        }
        .org-chart .level {
            display: flex;
            justify-content: center;
            align-items: flex-start;
            flex-wrap: wrap;
        }
        .org-chart .level + .level {
            margin-top: 2rem; /* Space between levels */
        }

        /* Modal Styles */
        .modal {
            position: fixed;
            z-index: 100;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
            background-color: rgba(0,0,0,0.4);
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 20px;
            border-radius: 8px;
            width: 80%;
            max-width: 600px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        .close-button {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
        }
        .close-button:hover,
        .close-button:focus {
            color: black;
            text-decoration: none;
            cursor: pointer;
        }
        /* Sorting indicator */
        th.sortable {
            cursor: pointer;
            position: relative;
        }
        th.sortable::after {
            content: '';
            position: absolute;
            right: 8px;
            top: 50%;
            transform: translateY(-50%);
            width: 0;
            height: 0;
            border-left: 5px solid transparent;
            border-right: 5px solid transparent;
            opacity: 0.4;
        }
        th.sortable.asc::after {
            border-bottom: 5px solid #000;
            margin-top: -5px;
        }
        th.sortable.desc::after {
            border-top: 5px solid #000;
            margin-top: 5px;
        }
    </style>
</head>
<body class="bg-slate-100">

    <div class="flex min-h-screen">
        <!-- Sidebar -->
        <aside class="w-64 bg-white text-gray-800 p-4 shadow-lg flex-shrink-0">
            <div class="text-center py-4">
                <h1 class="text-xl font-bold text-slate-800">ديوان الوقف الشيعي</h1>
                <p class="text-sm text-slate-500">قسم المحافظات الجنوبية</p>
            </div>
            <nav class="mt-8">
                <ul>
                    <li><a href="#dashboard" class="nav-link flex items-center p-3 my-1 bg-sky-100 text-sky-700 rounded-lg font-bold"><span class="sidebar-icon me-3">📊</span> لوحة التحكم</a></li>
                    <li><a href="#hr" class="nav-link flex items-center p-3 my-1 hover:bg-sky-100 hover:text-sky-700 rounded-lg font-semibold"><span class="sidebar-icon me-3">👥</span> الموارد البشرية</a></li>
                    <li><a href="#operations" class="nav-link flex items-center p-3 my-1 hover:bg-sky-100 hover:text-sky-700 rounded-lg font-semibold"><span class="sidebar-icon me-3">⚙️</span> العمليات</a></li>
                    <li><a href="#org-structure" class="nav-link flex items-center p-3 my-1 hover:bg-sky-100 hover:text-sky-700 rounded-lg font-semibold"><span class="sidebar-icon me-3">🏛️</span> الهيكل الإداري</a></li>
                    <li><a href="#directorates" class="nav-link flex items-center p-3 my-1 hover:bg-sky-100 hover:text-sky-700 rounded-lg font-semibold"><span class="sidebar-icon me-3">🏢</span> مديريات الوقف</a></li>
                    <li><a href="#reports" class="nav-link flex items-center p-3 my-1 hover:bg-sky-100 hover:text-sky-700 rounded-lg font-semibold"><span class="sidebar-icon me-3">📈</span> التقارير</a></li>
                </ul>
            </nav>
            <div class="mt-auto text-center pt-8">
                 <p class="text-xs text-gray-500">تصميم: ر. مهندسين</p>
                 <p class="text-sm font-semibold text-gray-700">رائد إبراهيم خليل</p>
            </div>
        </aside>

        <!-- Main Content -->
        <div class="flex-1 flex flex-col">
            <!-- Header -->
            <header class="bg-white shadow-sm p-4 flex justify-between items-center">
                <div id="header-title" class="text-xl font-bold text-slate-700">لوحة التحكم</div>
                <div class="flex items-center">
                    <span id="systemManagerName" class="text-sm font-semibold cursor-pointer hover:text-sky-700 transition-colors">مرحباً, مدير النظام</span>
                    <button id="logoutBtn" class="bg-red-500 text-white px-4 py-2 rounded-lg ms-4 hover:bg-red-600 transition-colors">خروج</button>
                </div>
            </header>

            <!-- Content Area -->
            <main class="flex-1 p-6 overflow-y-auto">
                <!-- Dashboard View -->
                <div id="dashboard-view" class="view">
                    <p class="mb-6 text-gray-600">هذه هي لوحة التحكم الرئيسية التي تعرض ملخصاً لأداء المديرية عبر منظورات بطاقة الأداء المتوازن الأربعة. كل بطاقة تقدم لمحة سريعة عن المؤشرات الرئيسية، مما يساعد على اتخاذ قرارات مستنيرة ومتابعة تحقيق الأهداف الاستراتيجية.</p>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                        <!-- Financial Perspective -->
                        <div class="bg-white p-6 rounded-xl shadow-md border-t-4 border-green-500">
                            <h3 class="font-bold text-lg text-green-700">المنظور المالي</h3>
                            <p class="text-sm text-gray-500 mb-4">كفاءة الإنفاق والاستدامة</p>
                            <div class="h-48 w-full flex items-center justify-center"><canvas id="financialChart"></canvas></div>
                            <p class="text-center mt-4 text-2xl font-bold text-gray-800">115% <span class="text-sm font-normal">من الهدف</span></p>
                        </div>
                        <!-- Customer Perspective -->
                        <div class="bg-white p-6 rounded-xl shadow-md border-t-4 border-sky-500">
                            <h3 class="font-bold text-lg text-sky-700">منظور المستفيدين</h3>
                            <p class="text-sm text-gray-500 mb-4">جودة الخدمة ورضا المتعاملين</p>
                            <div class="h-48 w-full flex items-center justify-center"><canvas id="customerChart"></canvas></div>
                            <p class="text-center mt-4 text-2xl font-bold text-gray-800">92% <span class="text-sm font-normal">معدل الرضا</span></p>
                        </div>
                        <!-- Internal Process Perspective -->
                        <div class="bg-white p-6 rounded-xl shadow-md border-t-4 border-amber-500">
                            <h3 class="font-bold text-lg text-amber-700">منظور العمليات الداخلية</h3>
                            <p class="text-sm text-gray-500 mb-4">كفاءة وسرعة الإجراءات</p>
                            <div class="h-48 w-full flex items-center justify-center"><canvas id="processChart"></canvas></div>
                             <p class="text-center mt-4 text-2xl font-bold text-gray-800">85% <span class="text-sm font-normal">أتمتة العمليات</span></p>
                        </div>
                        <!-- Learning & Growth Perspective -->
                        <div class="bg-white p-6 rounded-xl shadow-md border-t-4 border-purple-500">
                            <h3 class="font-bold text-lg text-purple-700">منظور التعلم والنمو</h3>
                            <p class="text-sm text-gray-500 mb-4">تطوير قدرات الموظفين</p>
                           <div class="h-48 w-full flex items-center justify-center"><canvas id="learningChart"></canvas></div>
                           <p class="text-center mt-4 text-2xl font-bold text-gray-800">75% <span class="text-sm font-normal">موظفين مدربين</span></p>
                        </div>
                    </div>
                </div>

                <!-- HR View -->
                <div id="hr-view" class="view hidden">
                    <p class="mb-6 text-gray-600">هذا القسم مخصص لإدارة جميع بيانات الموارد البشرية. يمكنك من خلاله عرض قائمة الموظفين، البحث عن موظف معين، والوصول إلى ملفه الشخصي الذي يحتوي على السيرة الذاتية، سجل الإجازات، المهام المكلف بها، وسجل نشاطاته اليومية. كما يمكنك إضافة، تعديل، وحذف بيانات الموظفين.</p>
                    <div class="bg-white p-6 rounded-xl shadow-md">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">قائمة الموظفين</h2>
                            <input type="text" id="employeeSearch" placeholder="ابحث عن موظف..." class="border p-2 rounded-lg">
                            <button id="addEmployeeBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة موظف جديد</button>
                        </div>
                        <div class="overflow-x-auto mb-6">
                            <table class="w-full text-right">
                                <thead class="border-b-2">
                                    <tr>
                                        <th class="p-3 sortable" data-sort-by="name">الاسم</th>
                                        <th class="p-3 sortable" data-sort-by="title">المنصب</th>
                                        <th class="p-3 sortable" data-sort-by="department">القسم</th>
                                        <th class="p-3 sortable" data-sort-by="hireDate">تاريخ التعيين</th>
                                        <th class="p-3">الإجراءات</th>
                                    </tr>
                                </thead>
                                <tbody id="employeeTableBody">
                                    <!-- Employee rows will be inserted here by JS -->
                                </tbody>
                            </table>
                        </div>

                        <!-- Add/Edit Employee Form -->
                        <div id="employeeFormSection" class="hidden bg-slate-50 p-6 rounded-lg shadow-inner">
                            <h3 id="formTitle" class="text-lg font-bold text-slate-700 mb-4">إضافة موظف جديد</h3>
                            <form id="employeeForm" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                                <div>
                                    <label for="employeeName" class="block text-sm font-medium text-gray-700">الاسم:</label>
                                    <input type="text" id="employeeName" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
                                </div>
                                <div>
                                    <label for="employeeTitle" class="block text-sm font-medium text-gray-700">المنصب:</label>
                                    <input type="text" id="employeeTitle" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
                                </div>
                                <div>
                                    <label for="employeeDepartment" class="block text-sm font-medium text-gray-700">القسم:</label>
                                    <input type="text" id="employeeDepartment" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
                                </div>
                                <div>
                                    <label for="employeeHireDate" class="block text-sm font-medium text-gray-700">تاريخ التعيين:</label>
                                    <input type="date" id="employeeHireDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
                                </div>
                                <div class="md:col-span-2">
                                    <label for="employeeCV" class="block text-sm font-medium text-gray-700">السيرة الذاتية والتحصيل الدراسي:</label>
                                    <textarea id="employeeCV" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
                                </div>
                                <div>
                                    <label for="employeeLeave" class="block text-sm font-medium text-gray-700">رصيد الإجازات:</label>
                                    <input type="number" id="employeeLeave" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
                                </div>
                                <div class="md:col-span-2 flex justify-end space-x-reverse space-x-4 mt-4">
                                    <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">حفظ</button>
                                    <button type="button" id="cancelFormBtn" class="bg-gray-300 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-400 transition-colors">إلغاء</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>

                <!-- Employee Profile View -->
                <div id="employee-profile-view" class="view hidden">
                    <!-- The back button to return to the HR employee list -->
                    <button id="back-to-hr" class="mb-4 bg-gray-200 px-4 py-2 rounded-lg hover:bg-gray-300">← العودة إلى قائمة الموظفين</button>
                    <div class="bg-white p-8 rounded-xl shadow-md">
                        <div id="profile-content">
                            <!-- Profile details will be inserted here by JS -->
                        </div>
                    </div>
                </div>

                <!-- Operations View -->
                <div id="operations-view" class="view hidden">
                    <p class="mb-6 text-gray-600">قسم العمليات هو المركز لمتابعة الأنشطة اليومية. هنا يمكنك إدارة المراسلات الواردة والصادرة، نشر وتوزيع الإعمامات الرسمية، وتخطيط ومتابعة جميع النشاطات والمبادرات التي تقوم بها المديرية لضمان سير العمل بسلاسة وفعالية.</p>
                    
                    <!-- Correspondence Section -->
                    <div class="bg-white p-6 rounded-xl shadow-md mb-6">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">إدارة المراسلات</h2>
                            <button id="addCorrespondenceBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة مراسلة جديدة</button>
                        </div>

                        <!-- Correspondence Tabs -->
                        <div class="flex border-b mb-4">
                            <button id="incomingCorrespondenceTab" class="py-2 px-4 text-sm font-medium text-sky-700 border-b-2 border-sky-500 focus:outline-none">مراسلات واردة</button>
                            <button id="outgoingCorrespondenceTab" class="py-2 px-4 text-sm font-medium text-gray-600 hover:text-sky-700 focus:outline-none">مراسلات صادرة</button>
                        </div>

                        <!-- Incoming Correspondence Table -->
                        <div id="incomingCorrespondenceTableContainer">
                            <div class="flex justify-between items-center mb-4">
                                <input type="text" id="incomingSearchInput" placeholder="بحث بالرقم، الموضوع، التاريخ، المرسل..." class="border p-2 rounded-lg flex-grow me-2">
                                <button id="incomingSearchBtn" class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition-colors">بحث</button>
                            </div>
                            <div class="overflow-x-auto">
                                <table class="w-full text-right">
                                    <thead class="border-b-2">
                                        <tr>
                                            <th class="p-3 sortable" data-sort-by="number">الرقم</th>
                                            <th class="p-3 sortable" data-sort-by="subject">الموضوع</th>
                                            <th class="p-3 sortable" data-sort-by="sender">المرسل</th>
                                            <th class="p-3 sortable" data-sort-by="date">التاريخ</th>
                                            <th class="p-3">الإجراءات</th>
                                        </tr>
                                    </thead>
                                    <tbody id="incomingCorrespondenceTableBody">
                                        <!-- Incoming Correspondence rows will be inserted here by JS -->
                                    </tbody>
                                </table>
                            </div>
                        </div>

                        <!-- Outgoing Correspondence Table -->
                        <div id="outgoingCorrespondenceTableContainer" class="hidden">
                            <div class="flex justify-between items-center mb-4">
                                <input type="text" id="outgoingSearchInput" placeholder="بحث بالرقم، الموضوع، التاريخ، المستلم..." class="border p-2 rounded-lg flex-grow me-2">
                                <button id="outgoingSearchBtn" class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition-colors">بحث</button>
                            </div>
                            <div class="overflow-x-auto">
                                <table class="w-full text-right">
                                    <thead class="border-b-2">
                                        <tr>
                                            <th class="p-3 sortable" data-sort-by="number">الرقم</th>
                                            <th class="p-3 sortable" data-sort-by="subject">الموضوع</th>
                                            <th class="p-3 sortable" data-sort-by="recipient">المستلم</th>
                                            <th class="p-3 sortable" data-sort-by="date">التاريخ</th>
                                            <th class="p-3">الإجراءات</th>
                                    </tr>
                                    </thead>
                                    <tbody id="outgoingCorrespondenceTableBody">
                                        <!-- Outgoing Correspondence rows will be inserted here by JS -->
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>

                    <!-- Announcements Section -->
                    <div class="bg-white p-6 rounded-xl shadow-md mb-6">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">إدارة الإعمامات</h2>
                            <button id="addAnnouncementBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة إعمام جديد</button>
                        </div>
                        <div class="flex justify-between items-center mb-4">
                            <input type="text" id="announcementSearchInput" placeholder="بحث بالعنوان، التاريخ، الجمهور المستهدف..." class="border p-2 rounded-lg flex-grow me-2">
                            <button id="announcementSearchBtn" class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition-colors">بحث</button>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-right">
                                <thead class="border-b-2">
                                    <tr>
                                        <th class="p-3 sortable" data-sort-by="title">العنوان</th>
                                        <th class="p-3 sortable" data-sort-by="date">التاريخ</th>
                                        <th class="p-3 sortable" data-sort-by="targetAudience">الجمهور المستهدف</th>
                                        <th class="p-3">الإجراءات</th>
                                    </tr>
                                </thead>
                                <tbody id="announcementTableBody">
                                    <!-- Announcement rows will be inserted here by JS -->
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <!-- Activities Section -->
                    <div class="bg-white p-6 rounded-xl shadow-md">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">إدارة النشاطات</h2>
                            <button id="addActivityBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة نشاط جديد</button>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-right">
                                <thead class="border-b-2">
                                    <tr>
                                        <th class="p-3 sortable" data-sort-by="name">اسم النشاط</th>
                                        <th class="p-3 sortable" data-sort-by="date">التاريخ</th>
                                        <th class="p-3 sortable" data-sort-by="status">الحالة</th>
                                        <th class="p-3">الإجراءات</th>
                                    </tr>
                                </thead>
                                <tbody id="activityTableBody">
                                    <!-- Activity rows will be inserted here by JS -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>

                <!-- Organizational Structure View -->
                <div id="org-structure-view" class="view hidden">
                    <p class="mb-6 text-gray-600">يعرض هذا القسم الهيكل الإداري لدائرة أوقاف المحافظات، موضحاً التسلسل الوظيفي والمسؤوليات الرئيسية لكل قسم. يساهم فهم هذا الهيكل في تبسيط سير العمل وتحسين التنسيق بين الأقسام المختلفة. انقر على أي مربع لعرض التفاصيل.</p>
                    <h2 class="text-xl font-bold text-slate-700 mb-6">الهيكل الإداري لدائرة أوقاف المحافظات</h2>
                    <div class="bg-white p-6 rounded-xl shadow-md org-chart overflow-x-auto pb-8">
                        <!-- Level 1: Director General -->
                        <div class="level">
                            <div class="node" data-org-unit-id="directorGeneral">المدير العام</div>
                        </div>

                        <!-- Level 2: Deputy and Office of Director General -->
                        <div class="level relative flex justify-center">
                            <div class="children single-child flex-col items-center">
                                <div class="node" data-org-unit-id="deputyDirector">معاون المدير العام</div>
                            </div>
                            <div class="children single-child flex-col items-center">
                                <div class="node" data-org-unit-id="directorGeneralOffice">مكتب المدير العام</div>
                                <div class="children flex-col items-center">
                                    <div class="node" data-org-unit-id="administrativeOffice">المكتب الإداري</div>
                                    <div class="node" data-org-unit-id="archivingOffice">الأرشفة</div>
                                </div>
                            </div>
                            <!-- Horizontal line connecting Deputy and Office of Director General children -->
                            <div class="absolute w-2/3 h-px bg-slate-400 top-0 left-1/2 transform -translate-x-1/2 -translate-y-1/2"></div>
                            <div class="absolute w-px h-6 bg-slate-400 top-0 left-[25%] transform -translate-x-1/2"></div>
                            <div class="absolute w-px h-6 bg-slate-400 top-0 left-[75%] transform -translate-x-1/2"></div>
                        </div>


                        <!-- Level 3: Regional Departments -->
                        <div class="level relative">
                            <div class="children flex flex-wrap justify-center w-full">
                                <div class="node" data-org-unit-id="centralProvinces">قسم المحافظات الوسطى</div>
                                <div class="node" data-org-unit-id="northernProvinces">قسم المحافظات الشمالية</div>
                                <div class="node" data-org-unit-id="southernProvinces">قسم المحافظات الجنوبية</div>
                                <div class="node" data-org-unit-id="middleEuphratesProvinces">قسم محافظات الفرات الأوسط</div>
                            </div>
                             <!-- Horizontal line connecting regional departments children -->
                            <div class="absolute w-full h-px bg-slate-400 top-0 left-0 transform -translate-y-1/2"></div>
                             <div class="absolute w-px h-6 bg-slate-400 top-0 left-[12.5%] transform -translate-x-1/2"></div>
                             <div class="absolute w-px h-6 bg-slate-400 top-0 left-[37.5%] transform -translate-x-1/2"></div>
                             <div class="absolute w-px h-6 bg-slate-400 top-0 left-[62.5%] transform -translate-x-1/2"></div>
                             <div class="absolute w-px h-6 bg-slate-400 top-0 left-[87.5%] transform -translate-x-1/2"></div>
                        </div>

                        <!-- Section: Responsibilities -->
                        <div class="mt-12">
                            <h3 class="text-lg font-bold text-slate-700 mb-4">مهام ومسؤوليات الدائرة والأقسام</h3>
                            <ul class="list-disc list-inside text-gray-700 space-y-2">
                                <li>**دائرة أوقاف المحافظات:** مسؤولة عن توجيه ومتابعة الإعمامات الصادرة من الجهات العليا إلى مديريات الديوان في المحافظات، متابعة بريدها ونشاطاتها، إعداد تقارير شهرية وفصلية وسنوية عنها، جرد أملاكها الموقوفة ومؤسساتها ومدارسها وجوامعها وكافة احتياجاتها، وحلحلة مشاكلها وتذليل صعوباتها.</li>
                                <li>**قسم المحافظات (الوسطى، الشمالية، الجنوبية، الفرات الأوسط):** مهامها الرئيسية هي الإشراف والتقييم للمديريات في كل المحافظات التابعة لها.</li>
                                <li>**الشعبة الإدارية (ضمن مكتب المدير العام):** مهمتها تحرير ومتابعة بريد المدير العام.</li>
                                <li>**الأرشفة (ضمن مكتب المدير العام):** مسؤولة عن أرشفة البريد الوارد والصادر وتوزيع البريد.</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <!-- Directorates View (New Section) -->
                <div id="directorates-view" class="view hidden">
                    <p class="mb-6 text-gray-600">يعرض هذا القسم نظرة عامة على مديريات الوقف الشيعي في المحافظات الجنوبية (البصرة، ميسان، ذي قار)، مع ملخص لأبرز الإحصائيات لكل مديرية. يمكنك إضافة مديريات جديدة أو تعديل وحذف المديريات الموجودة.</p>
                    <div class="bg-white p-6 rounded-xl shadow-md">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">مديريات الوقف الشيعي في المحافظات الجنوبية</h2>
                            <button id="addDirectorateBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة مديرية جديدة</button>
                        </div>
                        <div id="directoratesGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                            <!-- Directorate cards will be inserted here by JS -->
                        </div>
                        <div class="mt-6 p-4 bg-blue-50 border border-blue-200 rounded-lg text-blue-800">
                            <p class="font-semibold">ملاحظة:</p>
                            <p class="text-sm">هذه البيانات افتراضية. يمكن ربطها ببيانات حقيقية وتحديثها ديناميكياً في المستقبل.</p>
                        </div>
                    </div>
                </div>

                <!-- Reports View -->
                <div id="reports-view" class="view hidden">
                    <p class="mb-6 text-gray-600">يوفر هذا القسم تقارير تحليلية شاملة ومخصصة تساعد الإدارة على قياس الأداء، تحليل الاتجاهات، وضمان الامتثال للوائح، مما يدعم اتخاذ قرارات استراتيجية مبنية على بيانات دقيقة. كما يمكنك هنا تسجيل تقارير الزيارات للمتابعة وإجراء تحليلات شاملة للأداء.</p>
                    
                    <!-- Visit Reports Section -->
                    <div class="bg-white p-6 rounded-xl shadow-md mb-6">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">تقارير الزيارات</h2>
                            <button id="addVisitReportBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة تقرير زيارة</button>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-right">
                                <thead class="border-b-2">
                                    <tr>
                                        <th class="p-3 sortable" data-sort-by="date">التاريخ</th>
                                        <th class="p-3 sortable" data-sort-by="visitedEntity">الجهة المزارة</th>
                                        <th class="p-3 sortable" data-sort-by="purpose">الغرض</th>
                                        <th class="p-3">الإجراءات</th>
                                    </tr>
                                </thead>
                                <tbody id="visitReportTableBody">
                                    <!-- Visit Report rows will be inserted here by JS -->
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <!-- Comprehensive Performance Analysis Section -->
                    <div class="bg-white p-6 rounded-xl shadow-md mb-6">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-xl font-bold text-slate-700">تحليل الأداء الشامل</h2>
                             <button id="addPerformanceAnalysisBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة تقرير أداء جديد</button>
                        </div>
                        <p class="mb-4 text-gray-600">يُقدم هذا القسم تحليلاً معمقاً لأداء المديريات والشعب، مع التركيز على تحديد الإمكانات، العقبات، والاحتياجات. يتم استخدام أساليب تحليل بيانات حديثة لضمان تقارير دقيقة وشاملة تدعم اتخاذ القرار وتطوير الأداء المؤسسي.</p>
                        
                        <h3 class="text-lg font-bold text-slate-600 mb-3">تقييم الإمكانات والعقبات</h3>
                        <div class="overflow-x-auto mb-4">
                            <table class="w-full text-right border">
                                <thead class="bg-gray-50">
                                    <tr>
                                        <th class="p-3 sortable" data-sort-by="entity">المديرية/الشعبة</th>
                                        <th class="p-3 sortable" data-sort-by="capabilities">أبرز الإمكانات</th>
                                        <th class="p-3 sortable" data-sort-by="obstacles">التحديات/العقبات</th>
                                        <th class="p-3 sortable" data-sort-by="needs">الاحتياجات الرئيسية</th>
                                        <th class="p-3">الإجراءات</th>
                                    </tr>
                                </thead>
                                <tbody id="performanceAnalysisTableBody">
                                    <!-- Performance Analysis rows will be inserted here by JS -->
                                </tbody>
                            </table>
                        </div>

                        <h3 class="text-lg font-bold text-slate-600 mb-3">ملخص التوصيات والاستنتاجات</h3>
                        <div class="bg-slate-50 p-4 rounded-lg border border-slate-200">
                            <p class="text-gray-700 mb-2">تؤكد التحليلات على ضرورة تبني استراتيجيات مرنة لمواجهة التحديات المالية، مع الاستفادة القصوى من الكوادر البشرية وتطويرها المستمر. يوصى بإنشاء فرق عمل متخصصة لحل المشاكل المعقدة وتذليل العقبات الإدارية والفنية بتقنيات ذكية.</p>
                            <p class="text-gray-700">كما يجب التركيز على الرقمنة الشاملة للعمليات لزيادة الشفافية وتحسين سرعة الإنجاز، مع إعداد خطط تطوير مؤسسي وعلمي ومالي دورية لكل مديرية وشعبة بناءً على نتائج هذه التقارير.</p>
                        </div>
                    </div>
                </div>
            </main>
        </div>
    </div>

    <!-- Custom Confirmation Modal for Deletion -->
    <div id="confirmationModal" class="modal hidden">
        <div class="modal-content">
            <span class="close-button" id="closeModalBtn">&times;</span>
            <h3 class="text-xl font-bold text-slate-700 mb-4">تأكيد الحذف</h3>
            <p class="mb-6 text-gray-700">هل أنت متأكد أنك تريد حذف هذا العنصر؟ هذا الإجراء لا يمكن التراجع عنه.</p>
            <div class="flex justify-end space-x-reverse space-x-4">
                <button id="confirmDeleteBtn" class="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 transition-colors">تأكيد الحذف</button>
                <button id="cancelDeleteBtn" class="bg-gray-300 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-400 transition-colors">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- Generic Form Modal -->
    <div id="genericFormModal" class="modal hidden">
        <div class="modal-content">
            <span class="close-button" id="closeGenericFormModalBtn">&times;</span>
            <h3 id="genericFormTitle" class="text-xl font-bold text-slate-700 mb-4">عنوان النموذج</h3>
            <form id="genericForm" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <!-- Form fields will be dynamically inserted here -->
                <div id="modalFormFields" class="grid grid-cols-1 md:grid-cols-2 gap-4 md:col-span-2"></div>
                <div class="md:col-span-2 flex justify-end space-x-reverse space-x-4 mt-4">
                    <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">حفظ</button>
                    <button type="button" id="cancelGenericFormBtn" class="bg-gray-300 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-400 transition-colors">إلغاء</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Generic Detail Modal (now also used for Department Management) -->
    <div id="genericDetailModal" class="modal hidden">
        <div class="modal-content max-w-lg">
            <button id="detailModalBackBtn" class="mb-4 bg-gray-200 px-4 py-2 rounded-lg hover:bg-gray-300 text-sm">← العودة</button>
            <span class="close-button" id="closeGenericDetailModalBtn">&times;</span>
            <h3 id="detailModalTitle" class="text-xl font-bold text-slate-700 mb-4">عنوان التفاصيل</h3>
            <div id="detailModalContent" class="text-gray-700 space-y-3">
                <!-- Detailed content or Department list will be inserted here -->
            </div>
        </div>
    </div>

    <!-- System Manager Name Edit Modal -->
    <div id="managerNameModal" class="modal hidden">
        <div class="modal-content max-w-sm">
            <span class="close-button" id="closeManagerNameModalBtn">&times;</span>
            <h3 class="text-xl font-bold text-slate-700 mb-4">تعديل اسم مدير النظام</h3>
            <input type="text" id="newManagerNameInput" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500 mb-4" placeholder="أدخل الاسم الجديد">
            <div class="flex justify-end space-x-reverse space-x-4">
                <button id="saveManagerNameBtn" class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">حفظ</button>
                <button type="button" id="cancelManagerNameBtn" class="bg-gray-300 text-gray-800 px-4 py-2 rounded-lg hover:bg-gray-400 transition-colors">إلغاء</button>
            </div>
        </div>
    </div>


<script>
document.addEventListener('DOMContentLoaded', function () {
    const navLinks = document.querySelectorAll('.nav-link');
    const views = document.querySelectorAll('.view');
    const headerTitle = document.getElementById('header-title');
    const systemManagerNameSpan = document.getElementById('systemManagerName');
    const logoutBtn = document.getElementById('logoutBtn');
    const managerNameModal = document.getElementById('managerNameModal');
    const closeManagerNameModalBtn = document.getElementById('closeManagerNameModalBtn');
    const cancelManagerNameBtn = document.getElementById('cancelManagerNameBtn');
    const saveManagerNameBtn = document.getElementById('saveManagerNameBtn');
    const newManagerNameInput = document.getElementById('newManagerNameInput');

    // Load manager name from local storage or set default
    let managerName = localStorage.getItem('systemManagerName') || 'مدير النظام';
    systemManagerNameSpan.textContent = `مرحباً, ${managerName}`;

    // Update manager name function
    function updateManagerNameDisplay() {
        systemManagerNameSpan.textContent = `مرحباً, ${managerName}`;
    }

    // Event listener for manager name click
    systemManagerNameSpan.addEventListener('click', () => {
        newManagerNameInput.value = managerName;
        managerNameModal.classList.remove('hidden');
    });

    // Close manager name modal
    closeManagerNameModalBtn.addEventListener('click', () => {
        managerNameModal.classList.add('hidden');
    });
    cancelManagerNameBtn.addEventListener('click', () => {
        managerNameModal.classList.add('hidden');
    });

    // Save new manager name
    saveManagerNameBtn.addEventListener('click', () => {
        const newName = newManagerNameInput.value.trim();
        if (newName) {
            managerName = newName;
            localStorage.setItem('systemManagerName', newName);
            updateManagerNameDisplay();
            managerNameModal.classList.add('hidden');
        } else {
            console.error('اسم مدير النظام لا يمكن أن يكون فارغاً.');
        }
    });

    // Logout button functionality
    logoutBtn.addEventListener('click', () => {
        console.log('User logged out.');
        localStorage.removeItem('systemManagerName'); // Clear manager name on logout
        // In a real application, you would perform a server-side logout
        // For this demo, we'll just reload the page to reset the state
        window.location.reload(); 
    });

    // Helper function to get data from localStorage or use default
    function loadData(key, defaultData) {
        const data = localStorage.getItem(key);
        const parsedData = data ? JSON.parse(data) : defaultData;
        return parsedData;
    }

    // Helper function to save data to localStorage
    function saveData(key, data) {
        localStorage.setItem(key, JSON.stringify(data));
    }

    let employeeData = loadData('employeeData', [
        { id: 1, name: 'علي حسن', title: 'مهندس مدني', department: 'الهندسة', hireDate: '2018-05-20', cv: 'شهادة بكالوريوس في الهندسة المدنية، خبرة 5 سنوات في إدارة المشاريع.', leaveBalance: 22, tasks: ['الإشراف على مشروع الوقف أ', 'مراجعة المخططات الهندسية'], activities: [{date: '2025-06-16', log: 'زيارة ميدانية لموقع مشروع الوقف أ.'}, {date: '2025-06-15', log: 'اجتماع مع فريق العمل.'}] },
        { id: 2, name: 'فاطمة محمد', title: 'محاسب', department: 'المالية', hireDate: '2020-02-15', cv: 'شهادة بكالوريوس في المحاسبة، شهادة محاسب قانوني معتمد.', leaveBalance: 15, tasks: ['إعداد الميزانية الشهرية', 'متابعة الإيرادات'], activities: [{date: '2025-06-16', log: 'تدقيق فواتير المصروفات.'}] },
        { id: 3, name: 'أحمد ياسين', title: 'باحث شرعي', department: 'الشؤون الشرعية', hireDate: '2019-09-01', cv: 'ماجستير في الفقه وأصوله.', leaveBalance: 30, tasks: ['مراجعة حجج الوقف', 'الرد على الاستفسارات الشرعية'], activities: [{date: '2025-06-16', log: 'بحث في مسألة وقفية.'}] },
        { id: 4, name: 'زينب كريم', title: 'مسؤول إداري', department: 'الإدارة', hireDate: '2021-11-10', cv: 'دبلوم في إدارة المكاتب.', leaveBalance: 18, tasks: ['تنظيم المراسلات', 'أرشفة الملفات'], activities: [{date: '2025-06-16', log: 'تسجيل البريد الوارد والصادر.'}]}
    ]);

    let correspondenceData = loadData('correspondenceData', [
        { id: 1, number: '123/2025', subject: 'طلب صيانة وقف الزهراء', type: 'وارد', date: '2025-06-20', sender: 'مديرية أوقاف البصرة', status: 'قيد المراجعة', action: 'تحويل إلى قسم الهندسة', content: 'وصلنا طلب من مديرية أوقاف البصرة بخصوص صيانة شاملة لوقف الزهراء رقم 123. يرجى مراجعة المخططات الهندسية وتحديد المتطلبات.' },
        { id: 2, number: '456/2025', subject: 'تعميم رقم 15 بشأن إعداد الموازنة', type: 'صادر', date: '2025-06-18', recipient: 'جميع مديريات الأوقاف', status: 'تم الإرسال', action: 'متابعة الاستجابة', content: 'بناءً على توجيهات السيد رئيس الديوان، يرجى إعداد مشروع الموازنة للعام القادم وإرساله خلال شهر من تاريخه.' },
        { id: 3, number: '789/2025', subject: 'استفسار عن وقف جديد', type: 'وارد', date: '2025-06-25', sender: 'مواطن كريم', status: 'تم الرد', action: 'أرشفة', content: 'استفسار من مواطن بخصوص إجراءات تسجيل وقف جديد.' },
        { id: 4, number: '101/2025', subject: 'دعوة لحضور اجتماع', type: 'صادر', date: '2025-06-28', recipient: 'قسم الشؤون المالية', status: 'تم الإرسال', action: 'متابعة الحضور', content: 'دعوة لحضور اجتماع لمناقشة الموازنة التشغيلية.' }
    ]);

    let announcementData = loadData('announcementData', [
        { id: 1, title: 'فتح باب التقديم لدورات تطوير الموظفين', date: '2025-06-22', targetAudience: 'جميع الموظفين', content: 'تعلن المديرية عن فتح باب التقديم لدورات تطوير الموظفين في مجالات الإدارة المالية وتقنيات الأرشفة الحديثة. آخر موعد للتقديم 2025-07-05.' },
        { id: 2, title: 'توجيهات بشأن جرد الأصول الوقفية', date: '2025-06-19', targetAudience: 'رؤساء الأقسام ومدراء المديريات', content: 'يجب على جميع الأقسام والمديريات الشروع في جرد شامل لكافة الأصول الوقفية وتقديم التقارير المفصلة قبل نهاية شهر يوليو.' },
        { id: 3, title: 'إعلان عن عطلة رسمية', date: '2025-07-01', targetAudience: 'جميع الموظفين', content: 'بمناسبة عيد الأضحى المبارك، تعلن المديرية عن عطلة رسمية تبدأ من تاريخ 2025-07-05 ولمدة 4 أيام.' }
    ]);

    let activityData = loadData('activityData', [
        { id: 1, name: 'ورشة عمل حول إدارة الأوقاف', date: '2025-07-10', location: 'قاعة الاجتماعات الرئيسية', status: 'مخطط لها', description: 'ورشة عمل تستهدف الموظفين الجدد لتعريفهم بآليات إدارة الأوقاف وأهميتها في خدمة المجتمع.' },
        { id: 2, name: 'حملة صيانة وقف الإمام علي', date: '2025-06-05', location: 'محافظة النجف', status: 'منجز', description: 'تم إنجاز أعمال الصيانة الدورية لوقف الإمام علي في النجف، شملت ترميمات وإصلاحات للمبنى.' }
    ]);

    let visitReportData = loadData('visitReportData', [
        { id: 1, date: '2025-06-10', visitedEntity: 'مديرية أوقاف كربلاء', purpose: 'متابعة مشاريع الإعمار', visitCommittee: 'لجنة الإعمار', findings: 'تم الإطلاع على تقدم العمل في مشروع توسعة المسجد الحسيني وتبين وجود تأخير بسيط.', recommendations: 'ضرورة تسريع وتيرة العمل وتذليل العقبات الإدارية.' },
        { id: 2, date: '2025-05-25', visitedEntity: 'مدرسة الوقف الشيعي - بابل', purpose: 'جرد احتياجات المدرسة', visitCommittee: 'لجنة تقييم الاحتياجات', findings: 'المدرسة بحاجة ماسة إلى تحديث التجهيزات الصفية وبعض الإصلاحات في البنية التحتية.', recommendations: 'تقديم طلب موازنة طارئة لتغطية الاحتياجات العاجلة وتضمين احتياجاتها في خطة الموازنة القادمة.' }
    ]);

    let performanceAnalysisData = loadData('performanceAnalysisData', [
        { id: 1, entity: 'مديرية أوقاف البصرة', capabilities: 'كوادر مؤهلة، أصول وقفية متنوعة', obstacles: 'نقص التمويل، تحديات صيانة المباني القديمة', needs: 'موازنة أكبر للصيانة، تدريب على التقنيات الحديثة' },
        { id: 2, entity: 'شعبة تكنولوجيا المعلومات', capabilities: 'مهندسون ذوو خبرة، بنية تحتية جيدة', obstacles: 'تقادم بعض الأنظمة، الحاجة لتكامل البيانات', needs: 'تحديث البرمجيات، دورات في أمن المعلومات' }
    ]);

    // New data for Directorates - Initialized as empty array for manual entry
    // Each directorate now includes a 'departments' array
    let directoratesData = loadData('directoratesData', [
        { id: 1, name: 'مديرية أوقاف البصرة', location: 'البصرة', sections: 5, employees: 120, description: 'مديرية مسؤولة عن إدارة الأوقاف في محافظة البصرة.', departments: [
            { id: 101, name: 'شعبة الشؤون الإدارية', head: 'أحمد علي', employees: 15, tasks: 'إدارة شؤون الموظفين والمراسلات.', publicSchools: 5, religiousSchools: 2, mosques: 15, hussainiyas: 10, needs: 'صيانة مكيفات، تجديد أثاث.' },
            { id: 102, name: 'شعبة المالية', head: 'فاطمة الزهراء', employees: 10, tasks: 'إعداد الميزانيات ومتابعة الإيرادات والمصروفات.', publicSchools: 0, religiousSchools: 0, mosques: 0, hussainiyas: 0, needs: 'برنامج محاسبي جديد.' },
            { id: 103, name: 'شعبة الهندسة والصيانة', head: 'علي حسين', employees: 20, tasks: 'الإشراف على صيانة وتطوير الأبنية الوقفية.', publicSchools: 0, religiousSchools: 0, mosques: 0, hussainiyas: 0, needs: 'معدات صيانة حديثة، تدريب فنيين.' }
        ]},
        { id: 2, name: 'مديرية أوقاف ميسان', location: 'ميسان', sections: 3, employees: 80, description: 'مديرية مسؤولة عن إدارة الأوقاف في محافظة ميسان.', departments: [
            { id: 201, name: 'شعبة القانونية', head: 'زينب قاسم', employees: 5, tasks: 'تقديم الاستشارات القانونية ومتابعة القضايا.', publicSchools: 3, religiousSchools: 1, mosques: 8, hussainiyas: 5, needs: 'تحديث المكتبة القانونية.' }
        ]}
    ]);

    // Added specific details for organizational units, including top-level roles
    let orgUnitDetails = {
        directorGeneral: { name: 'المدير العام', description: 'المدير العام هو المسؤول الأول عن إدارة الدائرة ووضع الاستراتيجيات العامة وتوجيه العمل.', head: 'السيد/ عبد الله الموسوي', responsibilities: ['القيادة والإشراف العام', 'وضع السياسات والاستراتيجيات', 'تمثيل الدائرة'] },
        deputyDirector: { name: 'معاون المدير العام', description: 'يساعد معاون المدير العام المدير العام في مهامه، وينوب عنه في حال غيابه، ويشرف على عدد من الأقسام.', head: 'السيد/ فلاح الحسيني', responsibilities: ['المساعدة في القيادة', 'الإشراف على الأقسام التابعة', 'تنسيق العمل'] },
        directorGeneralOffice: { name: 'مكتب المدير العام', description: 'يتولى مكتب المدير العام إدارة وتنظيم جدول أعمال المدير العام وتنسيق الاتصالات.', head: 'السيدة/ نادية عبد الرحمن', responsibilities: ['تنظيم مواعيد المدير العام', 'تنسيق الاتصالات', 'إدارة الوثائق السرية'] },
        administrativeOffice: { name: 'المكتب الإداري', description: 'المكتب الإداري مسؤول عن تحرير ومتابعة بريد المدير العام وكافة الأعمال الإدارية واللوجستية.', head: 'السيد/ علي الجبوري', responsibilities: ['تحرير ومتابعة بريد المدير العام', 'تنظيم الملفات', 'الدعم الإداري'] },
        archivingOffice: { name: 'الأرشفة', description: 'شعبة الأرشفة مسؤولة عن أرشفة البريد الوارد والصادر وتوزيع البريد بكفاءة.', head: 'السيد/ خالد العامري', responsibilities: ['أرشفة الوثائق', 'توزيع البريد الوارد والصادر', 'تطبيق نظم الأرشفة الحديثة'] },
        centralProvinces: { name: 'قسم المحافظات الوسطى', description: 'يشرف هذا القسم على مديريات الأوقاف في المحافظات الوسطى ويقدم لها الدعم.', head: 'السيد/ ماجد فاضل', responsibilities: ['الإشراف على مديريات الوسطى', 'تقييم الأداء', 'حل مشاكل المديريات'] },
        northernProvinces: { name: 'قسم المحافظات الشمالية', description: 'يشرف هذا القسم على مديريات الأوقاف في المحافظات الشمالية ويقدم لها الدعم.', head: 'السيدة/ سارة عبد الكريم', responsibilities: ['الإشراف على مديريات الشمالية', 'تقييم الأداء', 'حل مشاكل المديريات'] },
        southernProvinces: { name: 'قسم المحافظات الجنوبية', description: 'يشرف هذا القسم على مديريات الأوقاف في المحافظات الجنوبية ويقدم لها الدعم.', head: 'السيد/ قيس الرشيد', responsibilities: ['الإشراف على مديريات الجنوبية', 'تقييم الأداء', 'حل مشاكل المديريات'] },
        middleEuphratesProvinces: { name: 'قسم محافظات الفرات الأوسط', description: 'يشرف هذا القسم على مديريات الأوقاف في محافظات الفرات الأوسط ويقدم لها الدعم.', head: 'السيد/ جعفر الصادق', responsibilities: ['الإشراف على مديريات الفرات الأوسط', 'تقييم الأداء', 'حل مشاكل المديريات'] },
    };

    let currentEmployeeId = null; 
    let currentCorrespondenceId = null;
    let currentAnnouncementId = null;
    let currentActivityId = null;
    let currentVisitReportId = null;
    let currentPerformanceAnalysisId = null; 
    let currentDirectorateId = null; // New variable for directorate ID
    let currentDepartmentId = null; // New variable for department ID

    // Correspondence tab state
    let currentCorrespondenceTab = 'incoming'; // Default tab

    // Generic Modal elements
    const genericFormModal = document.getElementById('genericFormModal');
    const closeGenericFormModalBtn = document.getElementById('closeGenericFormModalBtn');
    const cancelGenericFormBtn = document.getElementById('cancelGenericFormBtn');
    const genericFormTitle = document.getElementById('genericFormTitle');
    const genericForm = document.getElementById('genericForm');
    const modalFormFields = document.getElementById('modalFormFields');

    const genericDetailModal = document.getElementById('genericDetailModal');
    const closeGenericDetailModalBtn = document.getElementById('closeGenericDetailModalBtn');
    const detailModalBackBtn = document.getElementById('detailModalBackBtn'); 
    const detailModalTitle = document.getElementById('detailModalTitle');
    const detailModalContent = document.getElementById('detailModalContent');

    // HR Elements
    const employeeFormSection = document.getElementById('employeeFormSection');
    const employeeForm = document.getElementById('employeeForm');
    const formTitle = document.getElementById('formTitle');
    const employeeNameInput = document.getElementById('employeeName');
    const employeeTitleInput = document.getElementById('employeeTitle');
    const employeeDepartmentInput = document.getElementById('employeeDepartment');
    const employeeHireDateInput = document.getElementById('employeeHireDate');
    const employeeCVInput = document.getElementById('employeeCV');
    const employeeLeaveInput = document.getElementById('employeeLeave');
    const addEmployeeBtn = document.getElementById('addEmployeeBtn');
    const cancelFormBtn = document.getElementById('cancelFormBtn');

    // Confirmation Modal elements
    const confirmationModal = document.getElementById('confirmationModal');
    const closeModalBtn = document.getElementById('closeModalBtn');
    const confirmDeleteBtn = document.getElementById('confirmDeleteBtn');
    const cancelDeleteBtn = document.getElementById('cancelDeleteBtn');
    let itemToDelete = { type: null, id: null, callback: null, directorateId: null }; // Stores info about item to delete, added directorateId for departments

    // Search input elements
    const incomingSearchInput = document.getElementById('incomingSearchInput');
    const incomingSearchBtn = document.getElementById('incomingSearchBtn');
    const outgoingSearchInput = document.getElementById('outgoingSearchInput');
    const outgoingSearchBtn = document.getElementById('outgoingSearchBtn');
    const announcementSearchInput = document.getElementById('announcementSearchInput');
    const announcementSearchBtn = document.getElementById('announcementSearchBtn');

    // Sorting state variables for tables
    let currentSortColumn = {
        employee: { column: null, direction: 'asc' },
        incomingCorrespondence: { column: null, direction: 'asc' },
        outgoingCorrespondence: { column: null, direction: 'asc' },
        announcement: { column: null, direction: 'asc' },
        activity: { column: null, direction: 'asc' },
        visitReport: { column: null, direction: 'asc' },
        performanceAnalysis: { column: null, direction: 'asc' },
        department: { column: null, direction: 'asc' } // New sort state for departments
    };

    /**
     * Hides all views and shows the specified view.
     * Also hides all modals and specific form sections to ensure a clean state.
     * @param {string} viewId - The ID of the view to show (e.g., 'hr', 'dashboard').
     */
    function showView(viewId) {
        console.log(`showView called with: ${viewId}`); // Debug log
        views.forEach(view => {
            if (!view.classList.contains('hidden')) {
                console.log(`Hiding view: ${view.id}`); // Debug log
            }
            view.classList.add('hidden');
        });
        const activeView = document.getElementById(viewId + '-view');
        if (activeView) {
            activeView.classList.remove('hidden');
            console.log(`${viewId}-view is now visible.`); // Debug log
        } else {
            console.error(`Error: ${viewId}-view element not found.`); // Debug log
        }
        // Hide all specific form sections/modals when switching views
        employeeFormSection.classList.add('hidden');
        hideGenericFormModal();
        hideGenericDetailModal();
        hideConfirmationModal();
        managerNameModal.classList.add('hidden'); // Ensure manager name modal is hidden
        
        // Special handling for operations view to ensure correct correspondence tab is active
        if (viewId === 'operations') {
            switchCorrespondenceTab(currentCorrespondenceTab);
            populateAnnouncementTable();
            populateActivityTable();
        }
        // Special handling for HR view
        if (viewId === 'hr') {
            populateEmployeeTable();
        }
        // Special handling for Reports view
        if (viewId === 'reports') {
            populateVisitReportTable();
            populatePerformanceAnalysisTable();
        }
        // Special handling for Directorates view
        if (viewId === 'directorates') {
            populateDirectoratesView();
        }
    }

    navLinks.forEach(link => {
        link.addEventListener('click', function (e) {
            e.preventDefault();
            const viewId = this.getAttribute('href').substring(1);

            navLinks.forEach(l => {
                l.classList.remove('bg-sky-100', 'text-sky-700', 'font-bold');
                l.classList.add('font-semibold');
            });
            
            this.classList.add('bg-sky-100', 'text-sky-700', 'font-bold');
            this.classList.remove('font-semibold');

            headerTitle.textContent = this.textContent.replace(/[\u{1F4CA}\u{1F465}\u{2699}\u{1F3DB}\u{1F4C1}\u{1F4C8}\u{1F4C8}\u{1F3E2}]/gu, '').trim(); // Clean emoji from title
            showView(viewId);
        });
    });

    // --- Sorting Functionality ---
    function setupTableSorting(tableId, dataArray, populateFunction, sortStateKey) { 
        const table = document.querySelector(`#${tableId}`);
        if (!table) return;

        const headers = table.querySelectorAll('th.sortable');
        headers.forEach(header => {
            header.addEventListener('click', function() {
                const sortBy = this.dataset.sortBy;
                let direction = 'asc';

                // Reset all other headers
                headers.forEach(h => {
                    if (h !== this) {
                        h.classList.remove('asc', 'desc');
                    }
                });

                if (currentSortColumn[sortStateKey].column === sortBy) {
                    direction = currentSortColumn[sortStateKey].direction === 'asc' ? 'desc' : 'asc';
                }
                currentSortColumn[sortStateKey] = { column: sortBy, direction: direction };

                this.classList.remove('asc', 'desc'); // Remove existing
                this.classList.add(direction); // Add new

                const sortedData = [...dataArray].sort((a, b) => {
                    let valA = a[sortBy];
                    let valB = b[sortBy];

                    // Handle dates for sorting
                    if (sortBy.toLowerCase().includes('dateuploaded') || sortBy.toLowerCase().includes('date')) {
                        valA = new Date(valA);
                        valB = new Date(valB);
                    } else if (typeof valA === 'string') {
                        valA = valA.toLowerCase();
                        valB = valB.toLowerCase();
                    }

                    if (valA < valB) {
                        return direction === 'asc' ? -1 : 1;
                    }
                    if (valA > valB) {
                        return direction === 'asc' ? 1 : -1;
                    }
                    return 0;
                });
                populateFunction(sortedData); // Pass sorted data to populate function
            });
        });
    }


    // --- Employee Management (HR) Functions ---
    function populateEmployeeTable(data = employeeData, filter = '') {
        const tableBody = document.getElementById('employeeTableBody');
        tableBody.innerHTML = '';
        const filteredData = data.filter(emp => 
            emp.name.includes(filter) || emp.title.includes(filter) || emp.department.includes(filter)
        );

        filteredData.forEach(emp => {
            const row = document.createElement('tr');
            row.className = 'border-b hover:bg-gray-50';
            row.innerHTML = `
                <td class="p-3">${emp.name}</td>
                <td class="p-3">${emp.title}</td>
                <td class="p-3">${emp.department}</td>
                <td class="p-3">${emp.hireDate}</td>
                <td class="p-3 flex space-x-reverse space-x-2">
                    <button class="view-profile-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-id="${emp.id}">عرض</button>
                    <button class="edit-employee-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-id="${emp.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="employee" data-id="${emp.id}">حذف</button>
                </td>
            `;
            tableBody.appendChild(row);
        });

        // Event delegation for view/edit/delete buttons
        tableBody.querySelectorAll('.view-profile-btn').forEach(btn => btn.addEventListener('click', function() { showEmployeeProfile(parseInt(this.dataset.id)); }));
        tableBody.querySelectorAll('.edit-employee-btn').forEach(btn => btn.addEventListener('click', function() { openEmployeeForm(employeeData.find(emp => emp.id === parseInt(this.dataset.id))); }));
    }

    function showEmployeeProfile(employeeId) {
        const employee = employeeData.find(emp => emp.id === employeeId);
        if (!employee) return;

        const profileContent = document.getElementById('profile-content');
        
        let tasksHTML = (employee.tasks && employee.tasks.length > 0) ? employee.tasks.map(task => `<li class="mb-1">${task}</li>`).join('') : '<li class="text-gray-500">لا توجد مهام مكلفة.</li>';
        let activitiesHTML = (employee.activities && employee.activities.length > 0) ? employee.activities.map(act => `<div class="mb-2"><span class="font-bold text-sm text-gray-600">${act.date}:</span> ${act.log}</div>`).join('') : '<div class="text-gray-500">لا توجد نشاطات يومية مسجلة.</div>';

        profileContent.innerHTML = `
            <div class="text-center border-b pb-4 mb-4">
                <h2 class="text-3xl font-bold text-slate-800">${employee.name}</h2>
                <p class="text-lg text-slate-600">${employee.title} - ${employee.department}</p>
                <p class="text-sm text-gray-500">تاريخ التعيين: ${employee.hireDate}</p>
            </div>
            <div>
                <h3 class="font-bold text-xl mb-2 text-sky-700">السيرة الذاتية والتحصيل الدراسي</h3>
                <p class="mb-4 text-gray-700">${employee.cv || 'لا تتوفر سيرة ذاتية.'}</p>
                
                <h3 class="font-bold text-xl mb-2 text-sky-700">رصيد الإجازات</h3>
                <p class="mb-4 text-gray-700">${employee.leaveBalance !== undefined ? employee.leaveBalance : 'غير محدد'} يوم</p>

                <h3 class="font-bold text-xl mb-2 text-sky-700">المهام المكلف بها</h3>
                <ul class="list-disc list-inside mb-4 text-gray-700">${tasksHTML}</ul>

                <h3 class="font-bold text-xl mb-2 text-sky-700">سجل النشاطات اليومية</h3>
                <div class="border rounded-lg p-3 bg-slate-50">${activitiesHTML}</div>
            </div>
        `;
        showView('employee-profile');
    }

    addEmployeeBtn.addEventListener('click', () => openEmployeeForm());
    cancelFormBtn.addEventListener('click', () => employeeFormSection.classList.add('hidden'));

    employeeForm.addEventListener('submit', function(e) {
        e.preventDefault();
        saveEmployee();
    });

    function openEmployeeForm(employee = null) {
        employeeFormSection.classList.remove('hidden');
        const today = new Date().toISOString().split('T')[0]; // Get today's date in ISO-MM-DD format
        if (employee) {
            formTitle.textContent = 'تعديل بيانات الموظف';
            employeeNameInput.value = employee.name;
            employeeTitleInput.value = employee.title;
            employeeDepartmentInput.value = employee.department;
            employeeHireDateInput.value = employee.hireDate;
            employeeCVInput.value = employee.cv || '';
            employeeLeaveInput.value = employee.leaveBalance || 0;
            currentEmployeeId = employee.id;
        } else {
            formTitle.textContent = 'إضافة موظف جديد';
            employeeForm.reset();
            employeeHireDateInput.value = today; // Set default to today
            currentEmployeeId = null;
        }
    }

    function saveEmployee() {
        const name = employeeNameInput.value;
        const title = employeeTitleInput.value;
        const department = employeeDepartmentInput.value;
        const hireDate = employeeHireDateInput.value;
        const cv = employeeCVInput.value;
        const leaveBalance = parseInt(employeeLeaveInput.value) || 0;

        if (!name || !title || !department || !hireDate) {
            console.error('يرجى ملء جميع الحقول المطلوبة (الاسم، المنصب، القسم، تاريخ التعيين).');
            return;
        }

        if (currentEmployeeId) {
            const index = employeeData.findIndex(emp => emp.id === currentEmployeeId);
            if (index !== -1) {
                employeeData[index] = { ...employeeData[index], name, title, department, hireDate, cv, leaveBalance };
            }
        } else {
            const newId = employeeData.length > 0 ? Math.max(...employeeData.map(emp => emp.id)) + 1 : 1;
            employeeData.push({
                id: newId, name, title, department, hireDate, cv, leaveBalance, tasks: [], activities: []
            });
        }
        saveData('employeeData', employeeData); // Save to localStorage
        employeeFormSection.classList.add('hidden');
        populateEmployeeTable();
    }

    function deleteEmployee(id) {
        employeeData = employeeData.filter(emp => emp.id !== id);
        saveData('employeeData', employeeData); // Save to localStorage
        populateEmployeeTable();
    }

    // --- Operations (Correspondence, Announcements, Activities) Functions ---

    // Correspondence Tabs and Tables
    const incomingCorrespondenceTab = document.getElementById('incomingCorrespondenceTab');
    const outgoingCorrespondenceTab = document.getElementById('outgoingCorrespondenceTab');
    const incomingCorrespondenceTableContainer = document.getElementById('incomingCorrespondenceTableContainer');
    const outgoingCorrespondenceTableContainer = document.getElementById('outgoingCorrespondenceTableContainer');
    const incomingCorrespondenceTableBody = document.getElementById('incomingCorrespondenceTableBody');
    const outgoingCorrespondenceTableBody = document.getElementById('outgoingCorrespondenceTableBody');


    function switchCorrespondenceTab(tabType) {
        currentCorrespondenceTab = tabType;
        if (tabType === 'incoming') {
            incomingCorrespondenceTab.classList.add('border-sky-500', 'text-sky-700');
            incomingCorrespondenceTab.classList.remove('border-transparent', 'text-gray-600');
            outgoingCorrespondenceTab.classList.remove('border-sky-500', 'text-sky-700');
            outgoingCorrespondenceTab.classList.add('border-transparent', 'text-gray-600');
            incomingCorrespondenceTableContainer.classList.remove('hidden');
            outgoingCorrespondenceTableContainer.classList.add('hidden');
            populateCorrespondenceTable('incoming', incomingSearchInput.value); // Pass current search query
            // Reset sort indicators for outgoing table when switching to incoming
            document.querySelectorAll('#outgoingCorrespondenceTableContainer th.sortable').forEach(th => th.classList.remove('asc', 'desc'));
            currentSortColumn.outgoingCorrespondence = { column: null, direction: 'asc' };
        } else { // outgoing
            outgoingCorrespondenceTab.classList.add('border-sky-500', 'text-sky-700');
            outgoingCorrespondenceTab.classList.remove('border-transparent', 'text-gray-600');
            incomingCorrespondenceTab.classList.remove('border-sky-500', 'text-sky-700');
            incomingCorrespondenceTab.classList.add('border-transparent', 'text-gray-600');
            outgoingCorrespondenceTableContainer.classList.remove('hidden');
            incomingCorrespondenceTableContainer.classList.add('hidden');
            populateCorrespondenceTable('outgoing', outgoingSearchInput.value); // Pass current search query
            // Reset sort indicators for incoming table when switching to outgoing
            document.querySelectorAll('#incomingCorrespondenceTableContainer th.sortable').forEach(th => th.classList.remove('asc', 'desc'));
            currentSortColumn.incomingCorrespondence = { column: null, direction: 'asc' };
        }
    }

    // Initial tab selection
    switchCorrespondenceTab('incoming');

    incomingCorrespondenceTab.addEventListener('click', () => switchCorrespondenceTab('incoming'));
    outgoingCorrespondenceTab.addEventListener('click', () => switchCorrespondenceTab('outgoing'));

    // Search event listeners for correspondence
    incomingSearchBtn.addEventListener('click', () => populateCorrespondenceTable('incoming', incomingSearchInput.value));
    incomingSearchInput.addEventListener('keyup', (e) => { if (e.key === 'Enter') populateCorrespondenceTable('incoming', incomingSearchInput.value); });

    outgoingSearchBtn.addEventListener('click', () => populateCorrespondenceTable('outgoing', outgoingSearchInput.value));
    outgoingSearchInput.addEventListener('keyup', (e) => { if (e.key === 'Enter') populateCorrespondenceTable('outgoing', outgoingSearchInput.value); });


    function populateCorrespondenceTable(type, filter = '', data = correspondenceData) {
        const tableBody = type === 'incoming' ? incomingCorrespondenceTableBody : outgoingCorrespondenceTableBody;
        tableBody.innerHTML = '';
        const lowerCaseFilter = filter.toLowerCase();
        
        const filteredData = data.filter(mail => 
            mail.type === (type === 'incoming' ? 'وارد' : 'صادر') &&
            (String(mail.number).toLowerCase().includes(lowerCaseFilter) || // Search by number
             mail.subject.toLowerCase().includes(lowerCaseFilter) ||
             mail.date.includes(lowerCaseFilter) ||
             (mail.sender && mail.sender.toLowerCase().includes(lowerCaseFilter)) ||
             (mail.recipient && mail.recipient.toLowerCase().includes(lowerCaseFilter)))
        );

        filteredData.forEach(mail => {
            const row = document.createElement('tr');
            row.className = 'border-b hover:bg-gray-50';
            const senderRecipientText = type === 'incoming' ? mail.sender : mail.recipient;
            row.innerHTML = `
                <td class="p-3">${mail.number || 'N/A'}</td>
                <td class="p-3">${mail.subject}</td>
                <td class="p-3">${senderRecipientText || 'غير محدد'}</td>
                <td class="p-3">${mail.date}</td>
                <td class="p-3 flex space-x-reverse space-x-2">
                    <button class="view-item-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-type="correspondence" data-id="${mail.id}">عرض</button>
                    <button class="edit-item-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-type="correspondence" data-id="${mail.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="correspondence" data-id="${mail.id}">حذف</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
        // Event delegation for view/edit/delete buttons
        tableBody.querySelectorAll('.view-item-btn[data-type="correspondence"]').forEach(btn => btn.addEventListener('click', function() { viewCorrespondenceDetails(parseInt(this.dataset.id)); }));
        tableBody.querySelectorAll('.edit-item-btn[data-type="correspondence"]').forEach(btn => btn.addEventListener('click', function() { openCorrespondenceForm(correspondenceData.find(m => m.id === parseInt(this.dataset.id))); }));
    }

    document.getElementById('addCorrespondenceBtn').addEventListener('click', () => openCorrespondenceForm());

    function openCorrespondenceForm(mail = null) {
        genericFormTitle.textContent = mail ? 'تعديل بيانات المراسلة' : 'إضافة مراسلة جديدة';
        const today = new Date().toISOString().split('T')[0]; // Get today's date in ISO-MM-DD format
        modalFormFields.innerHTML = `
            <div>
                <label for="mailNumber" class="block text-sm font-medium text-gray-700">الرقم:</label>
                <input type="text" id="mailNumber" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="mailSubject" class="block text-sm font-medium text-gray-700">الموضوع:</label>
                <input type="text" id="mailSubject" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="mailType" class="block text-sm font-medium text-gray-700">النوع:</label>
                <select id="mailType" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
                    <option value="وارد">وارد</option>
                    <option value="صادر">صادر</option>
                </select>
            </div>
            <div>
                <label for="mailDate" class="block text-sm font-medium text-gray-700">التاريخ:</label>
                <input type="date" id="mailDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="mailSenderRecipient" class="block text-sm font-medium text-gray-700">المرسل/المستلم:</label>
                <input type="text" id="mailSenderRecipient" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="mailStatus" class="block text-sm font-medium text-gray-700">الحالة:</label>
                <input type="text" id="mailStatus" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="mailAction" class="block text-sm font-medium text-gray-700">الإجراء المطلوب:</label>
                <input type="text" id="mailAction" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div class="md:col-span-2">
                <label for="mailContent" class="block text-sm font-medium text-gray-700">المحتوى/الملاحظات:</label>
                <textarea id="mailContent" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
        `;
        if (mail) {
            document.getElementById('mailNumber').value = mail.number || '';
            document.getElementById('mailSubject').value = mail.subject;
            document.getElementById('mailType').value = mail.type;
            document.getElementById('mailDate').value = mail.date;
            document.getElementById('mailSenderRecipient').value = mail.sender || mail.recipient || '';
            document.getElementById('mailStatus').value = mail.status || '';
            document.getElementById('mailAction').value = mail.action || '';
            document.getElementById('mailContent').value = mail.content || '';
            currentCorrespondenceId = mail.id;
        } else {
            genericForm.reset();
            // Pre-select mail type based on active tab
            document.getElementById('mailType').value = (currentCorrespondenceTab === 'incoming' ? 'وارد' : 'صادر');
            document.getElementById('mailDate').value = today; // Set default to today
            currentCorrespondenceId = null;
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); saveCorrespondence(); };
    }

    function saveCorrespondence() {
        const number = document.getElementById('mailNumber').value;
        const subject = document.getElementById('mailSubject').value;
        const type = document.getElementById('mailType').value;
        const date = document.getElementById('mailDate').value;
        const senderRecipient = document.getElementById('mailSenderRecipient').value;
        const status = document.getElementById('mailStatus').value;
        const action = document.getElementById('mailAction').value;
        const content = document.getElementById('mailContent').value;

        if (!subject || !type || !date) {
            console.error('يرجى ملء جميع الحقول المطلوبة للمراسلة.');
            return;
        }

        if (currentCorrespondenceId) {
            const index = correspondenceData.findIndex(m => m.id === currentCorrespondenceId);
            if (index !== -1) {
                correspondenceData[index] = { ...correspondenceData[index], number, subject, type, date, sender: (type === 'وارد' ? senderRecipient : ''), recipient: (type === 'صادر' ? senderRecipient : ''), status, action, content };
            }
        } else {
            const newId = correspondenceData.length > 0 ? Math.max(...correspondenceData.map(m => m.id)) + 1 : 1;
            correspondenceData.push({ id: newId, number, subject, type, date, sender: (type === 'وارد' ? senderRecipient : ''), recipient: (type === 'صادر' ? senderRecipient : ''), status, action, content });
        }
        saveData('correspondenceData', correspondenceData); // Save to localStorage
        hideGenericFormModal();
        populateCorrespondenceTable(currentCorrespondenceTab); // Re-populate current active tab
    }

    function deleteCorrespondence(id) {
        correspondenceData = correspondenceData.filter(m => m.id !== id);
        saveData('correspondenceData', correspondenceData); // Save to localStorage
        populateCorrespondenceTable(currentCorrespondenceTab); // Re-populate current active tab
    }

    function viewCorrespondenceDetails(id) {
        const mail = correspondenceData.find(m => m.id === id);
        if (!mail) return;
        detailModalTitle.textContent = `تفاصيل المراسلة: ${mail.subject}`;
        detailModalContent.innerHTML = `
            <p><strong class="text-sky-700">الرقم:</strong> ${mail.number || 'غير محدد'}</p>
            <p><strong class="text-sky-700">الموضوع:</strong> ${mail.subject}</p>
            <p><strong class="text-sky-700">النوع:</strong> ${mail.type}</p>
            <p><strong class="text-sky-700">التاريخ:</strong> ${mail.date}</p>
            <p><strong class="text-sky-700">${mail.type === 'وارد' ? 'المرسل' : 'المستلم'}:</strong> ${mail.sender || mail.recipient || 'غير محدد'}</p>
            <p><strong class="text-sky-700">الحالة:</strong> ${mail.status || 'غير محدد'}</p>
            <p><strong class="text-sky-700">الإجراء المطلوب:</strong> ${mail.action || 'لا يوجد'}</p>
            <p><strong class="text-sky-700">المحتوى/الملاحظات:</strong> ${mail.content || 'لا يوجد محتوى.'}</p>
        `;
        genericDetailModal.classList.remove('hidden');
    }

    // Announcements
    // Search event listener for announcements
    announcementSearchBtn.addEventListener('click', () => populateAnnouncementTable(announcementSearchInput.value));
    announcementSearchInput.addEventListener('keyup', (e) => { if (e.key === 'Enter') populateAnnouncementTable(announcementSearchInput.value); });

    function populateAnnouncementTable(data = announcementData, filter = '') {
        const tableBody = document.getElementById('announcementTableBody');
        tableBody.innerHTML = '';
        const lowerCaseFilter = filter.toLowerCase();

        const filteredData = data.filter(ann => 
            ann.title.toLowerCase().includes(lowerCaseFilter) ||
            ann.date.includes(lowerCaseFilter) ||
            ann.targetAudience.toLowerCase().includes(lowerCaseFilter)
        );

        filteredData.forEach(ann => {
            const row = document.createElement('tr');
            row.className = 'border-b hover:bg-gray-50';
            row.innerHTML = `
                <td class="p-3">${ann.title}</td>
                <td class="p-3">${ann.date}</td>
                <td class="p-3">${ann.targetAudience}</td>
                <td class="p-3 flex space-x-reverse space-x-2">
                    <button class="view-item-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-type="announcement" data-id="${ann.id}">عرض</button>
                    <button class="edit-item-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-type="announcement" data-id="${ann.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="announcement" data-id="${ann.id}">حذف</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
        tableBody.querySelectorAll('.view-item-btn[data-type="announcement"]').forEach(btn => btn.addEventListener('click', function() { viewAnnouncementDetails(parseInt(this.dataset.id)); }));
        tableBody.querySelectorAll('.edit-item-btn[data-type="announcement"]').forEach(btn => btn.addEventListener('click', function() { openAnnouncementForm(announcementData.find(a => a.id === parseInt(this.dataset.id))); }));
    }

    document.getElementById('addAnnouncementBtn').addEventListener('click', () => openAnnouncementForm());

    function openAnnouncementForm(announcement = null) {
        genericFormTitle.textContent = announcement ? 'تعديل الإعمام' : 'إضافة إعمام جديد';
        const today = new Date().toISOString().split('T')[0]; // Get today's date in ISO-MM-DD format
        modalFormFields.innerHTML = `
            <div>
                <label for="annTitle" class="block text-sm font-medium text-gray-700">العنوان:</label>
                <input type="text" id="annTitle" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="annDate" class="block text-sm font-medium text-gray-700">التاريخ:</label>
                <input type="date" id="annDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div class="md:col-span-2">
                <label for="annTargetAudience" class="block text-sm font-medium text-gray-700">الجمهور المستهدف:</label>
                <input type="text" id="annTargetAudience" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div class="md:col-span-2">
                <label for="annContent" class="block text-sm font-medium text-gray-700">المحتوى:</label>
                <textarea id="annContent" rows="4" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required></textarea>
            </div>
        `;
        if (announcement) {
            document.getElementById('annTitle').value = announcement.title;
            document.getElementById('annDate').value = announcement.date;
            document.getElementById('annTargetAudience').value = announcement.targetAudience || '';
            document.getElementById('annContent').value = announcement.content;
            currentAnnouncementId = announcement.id;
        } else {
            genericForm.reset();
            document.getElementById('annDate').value = today; // Set default to today
            currentAnnouncementId = null;
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); saveAnnouncement(); };
    }

    function saveAnnouncement() {
        const title = document.getElementById('annTitle').value;
        const date = document.getElementById('annDate').value;
        const targetAudience = document.getElementById('annTargetAudience').value;
        const content = document.getElementById('annContent').value;

        if (!title || !date || !content) {
            console.error('يرجى ملء جميع الحقول المطلوبة للإعمام.');
            return;
        }

        if (currentAnnouncementId) {
            const index = announcementData.findIndex(a => a.id === currentAnnouncementId);
            if (index !== -1) {
                announcementData[index] = { ...announcementData[index], title, date, targetAudience, content };
            }
        } else {
            const newId = announcementData.length > 0 ? Math.max(...announcementData.map(a => a.id)) + 1 : 1;
            announcementData.push({ id: newId, title, date, targetAudience, content });
        }
        saveData('announcementData', announcementData); // Save to localStorage
        hideGenericFormModal();
        populateAnnouncementTable();
    }

    function deleteAnnouncement(id) {
        announcementData = announcementData.filter(a => a.id !== id);
        saveData('announcementData', announcementData); // Save to localStorage
        populateAnnouncementTable();
    }

    function viewAnnouncementDetails(id) {
        const ann = announcementData.find(a => a.id === id);
        if (!ann) return;
        detailModalTitle.textContent = `تفاصيل الإعمام: ${ann.title}`;
        detailModalContent.innerHTML = `
            <p><strong class="text-sky-700">العنوان:</strong> ${ann.title}</p>
            <p><strong class="text-sky-700">التاريخ:</strong> ${ann.date}</p>
            <p><strong class="text-sky-700">الجمهور المستهدف:</strong> ${ann.targetAudience || 'غير محدد'}</p>
            <p><strong class="text-sky-700">المحتوى:</strong> ${ann.content}</p>
        `;
        genericDetailModal.classList.remove('hidden');
    }

    // Activities
    function populateActivityTable(data = activityData) {
        const tableBody = document.getElementById('activityTableBody');
        tableBody.innerHTML = '';
        data.forEach(act => {
            const row = document.createElement('tr');
            row.className = 'border-b hover:bg-gray-50';
            row.innerHTML = `
                <td class="p-3">${act.name}</td>
                <td class="p-3">${act.date}</td>
                <td class="p-3">${act.status}</td>
                <td class="p-3 flex space-x-reverse space-x-2">
                    <button class="view-item-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-type="activity" data-id="${act.id}">عرض</button>
                    <button class="edit-item-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-type="activity" data-id="${act.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="activity" data-id="${act.id}">حذف</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
        tableBody.querySelectorAll('.view-item-btn[data-type="activity"]').forEach(btn => btn.addEventListener('click', function() { viewActivityDetails(parseInt(this.dataset.id)); }));
        tableBody.querySelectorAll('.edit-item-btn[data-type="activity"]').forEach(btn => btn.addEventListener('click', function() { openActivityForm(activityData.find(a => a.id === parseInt(this.dataset.id))); }));
    }

    document.getElementById('addActivityBtn').addEventListener('click', () => openActivityForm());

    function openActivityForm(activity = null) {
        genericFormTitle.textContent = activity ? 'تعديل النشاط' : 'إضافة نشاط جديد';
        const today = new Date().toISOString().split('T')[0]; // Get today's date in ISO-MM-DD format
        modalFormFields.innerHTML = `
            <div>
                <label for="activityName" class="block text-sm font-medium text-gray-700">اسم النشاط:</label>
                <input type="text" id="activityName" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="activityDate" class="block text-sm font-medium text-gray-700">التاريخ:</label>
                <input type="date" id="activityDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="activityLocation" class="block text-sm font-medium text-gray-700">الموقع:</label>
                <input type="text" id="activityLocation" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="activityStatus" class="block text-sm font-medium text-gray-700">الحالة:</label>
                <input type="text" id="activityStatus" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div class="md:col-span-2">
                <label for="activityDescription" class="block text-sm font-medium text-gray-700">الوصف:</label>
                <textarea id="activityDescription" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
        `;
        if (activity) {
            document.getElementById('activityName').value = activity.name;
            document.getElementById('activityDate').value = activity.date;
            document.getElementById('activityLocation').value = activity.location || '';
            document.getElementById('activityStatus').value = activity.status || '';
            document.getElementById('activityDescription').value = activity.description || '';
            currentActivityId = activity.id;
        } else {
            genericForm.reset();
            document.getElementById('activityDate').value = today; // Set default to today
            currentActivityId = null;
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); saveActivity(); };
    }

    function saveActivity() {
        const name = document.getElementById('activityName').value;
        const date = document.getElementById('activityDate').value;
        const location = document.getElementById('activityLocation').value;
        const status = document.getElementById('activityStatus').value;
        const description = document.getElementById('activityDescription').value;

        if (!name || !date) {
            console.error('يرجى ملء جميع الحقول المطلوبة للنشاط.');
            return;
        }

        if (currentActivityId) {
            const index = activityData.findIndex(a => a.id === currentActivityId);
            if (index !== -1) {
                activityData[index] = { ...activityData[index], name, date, location, status, description };
            }
        } else {
            const newId = activityData.length > 0 ? Math.max(...activityData.map(a => a.id)) + 1 : 1;
            activityData.push({ id: newId, name, date, location, status, description });
        }
        saveData('activityData', activityData); // Save to localStorage
        hideGenericFormModal();
        populateActivityTable();
    }

    function deleteActivity(id) {
        activityData = activityData.filter(a => a.id !== id);
        saveData('activityData', activityData); // Save to localStorage
        populateActivityTable();
    }

    function viewActivityDetails(id) {
        const act = activityData.find(a => a.id === id);
        if (!act) return;
        detailModalTitle.textContent = `تفاصيل النشاط: ${act.name}`;
        detailModalContent.innerHTML = `
            <p><strong class="text-sky-700">اسم النشاط:</strong> ${act.name}</p>
            <p><strong class="text-sky-700">التاريخ:</strong> ${act.date}</p>
            <p><strong class="text-sky-700">الموقع:</strong> ${act.location || 'غير محدد'}</p>
            <p><strong class="text-sky-700">الحالة:</strong> ${act.status || 'غير محدد'}</p>
            <p><strong class="text-sky-700">الوصف:</strong> ${act.description || 'لا يوجد وصف.'}</p>
        `;
        genericDetailModal.classList.remove('hidden');
    }

    // --- Reports (Visit Reports) Functions ---
    function populateVisitReportTable(data = visitReportData) {
        const tableBody = document.getElementById('visitReportTableBody');
        tableBody.innerHTML = '';
        data.forEach(report => {
            const row = document.createElement('tr');
            row.className = 'border-b hover:bg-gray-50';
            row.innerHTML = `
                <td class="p-3">${report.date}</td>
                <td class="p-3">${report.visitedEntity}</td>
                <td class="p-3">${report.purpose}</td>
                <td class="p-3 flex space-x-reverse space-x-2">
                    <button class="view-item-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-type="visitReport" data-id="${report.id}">عرض</button>
                    <button class="edit-item-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-type="visitReport" data-id="${report.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="visitReport" data-id="${report.id}">حذف</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
        tableBody.querySelectorAll('.view-item-btn[data-type="visitReport"]').forEach(btn => btn.addEventListener('click', function() { viewVisitReportDetails(parseInt(this.dataset.id)); }));
        tableBody.querySelectorAll('.edit-item-btn[data-type="visitReport"]').forEach(btn => btn.addEventListener('click', function() { openVisitReportForm(visitReportData.find(r => r.id === parseInt(this.dataset.id))); }));
    }

    document.getElementById('addVisitReportBtn').addEventListener('click', () => openVisitReportForm());

    function openVisitReportForm(report = null) {
        genericFormTitle.textContent = report ? 'تعديل تقرير الزيارة' : 'إضافة تقرير زيارة جديد';
        const today = new Date().toISOString().split('T')[0]; // Get today's date in ISO-MM-DD format
        modalFormFields.innerHTML = `
            <div>
                <label for="reportDate" class="block text-sm font-medium text-gray-700">التاريخ:</label>
                <input type="date" id="reportDate" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="reportVisitedEntity" class="block text-sm font-medium text-gray-700">الجهة المزارة:</label>
                <input type="text" id="reportVisitedEntity" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="reportVisitCommittee" class="block text-sm font-medium text-gray-700">لجنة الزيارة:</label>
                <input type="text" id="reportVisitCommittee" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div class="md:col-span-2">
                <label for="reportPurpose" class="block text-sm font-medium text-gray-700">الغرض من الزيارة:</label>
                <textarea id="reportPurpose" rows="2" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required></textarea>
            </div>
            <div class="md:col-span-2">
                <label for="reportFindings" class="block text-sm font-medium text-gray-700">النتائج/الملاحظات:</label>
                <textarea id="reportFindings" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
            <div class="md:col-span-2">
                <label for="reportRecommendations" class="block text-sm font-medium text-gray-700">التوصيات:</label>
                <textarea id="reportRecommendations" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
        `;
        if (report) {
            document.getElementById('reportDate').value = report.date;
            document.getElementById('reportVisitedEntity').value = report.visitedEntity;
            document.getElementById('reportPurpose').value = report.purpose;
            document.getElementById('reportVisitCommittee').value = report.visitCommittee || ''; // Populate new field
            document.getElementById('reportFindings').value = report.findings || '';
            document.getElementById('reportRecommendations').value = report.recommendations || '';
            currentVisitReportId = report.id;
        } else {
            genericForm.reset();
            document.getElementById('reportDate').value = today; // Set default to today
            currentVisitReportId = null;
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); saveVisitReport(); };
    }

    function saveVisitReport() {
        const date = document.getElementById('reportDate').value;
        const visitedEntity = document.getElementById('reportVisitedEntity').value;
        const purpose = document.getElementById('reportPurpose').value;
        const visitCommittee = document.getElementById('reportVisitCommittee').value; // Get new field value
        const findings = document.getElementById('reportFindings').value;
        const recommendations = document.getElementById('reportRecommendations').value;

        if (!date || !visitedEntity || !purpose) {
            console.error('يرجى ملء جميع الحقول المطلوبة لتقرير الزيارة.');
            return;
        }

        if (currentVisitReportId) {
            const index = visitReportData.findIndex(r => r.id === currentVisitReportId);
            if (index !== -1) {
                visitReportData[index] = { ...visitReportData[index], date, visitedEntity, purpose, visitCommittee, findings, recommendations };
            }
        } else {
            const newId = visitReportData.length > 0 ? Math.max(...visitReportData.map(r => r.id)) + 1 : 1;
            visitReportData.push({ id: newId, date, visitedEntity, purpose, visitCommittee, findings, recommendations });
        }
        saveData('visitReportData', visitReportData); // Save to localStorage
        hideGenericFormModal();
        populateVisitReportTable();
    }

    function deleteVisitReport(id) {
        visitReportData = visitReportData.filter(r => r.id !== id);
        saveData('visitReportData', visitReportData); // Save to localStorage
        populateVisitReportTable();
    }

    function viewVisitReportDetails(id) {
        const report = visitReportData.find(r => r.id === id);
        if (!report) return;
        detailModalTitle.textContent = `تفاصيل تقرير الزيارة: ${report.visitedEntity}`;
        detailModalContent.innerHTML = `
            <p><strong class="text-sky-700">التاريخ:</strong> ${report.date}</p>
            <p><strong class="text-sky-700">الجهة المزارة:</strong> ${report.visitedEntity}</p>
            <p><strong class="text-sky-700">الغرض من الزيارة:</strong> ${report.purpose}</p>
            <p><strong class="text-sky-700">لجنة الزيارة:</strong> ${report.visitCommittee || 'لا يوجد.'}</p>
            <p><strong class="text-sky-700">النتائج/الملاحظات:</strong> ${report.findings || 'لا يوجد.'}</p>
            <p><strong class="text-sky-700">التوصيات:</strong> ${report.recommendations || 'لا يوجد.'}</p>
        `;
        genericDetailModal.classList.remove('hidden');
        detailModalBackBtn.dataset.returnTo = 'reports'; // Set return view
    }

    // --- Performance Analysis Functions ---
    function populatePerformanceAnalysisTable(data = performanceAnalysisData) {
        const tableBody = document.getElementById('performanceAnalysisTableBody');
        tableBody.innerHTML = '';
        data.forEach(item => {
            const row = document.createElement('tr');
            row.className = 'border-b hover:bg-gray-50';
            row.innerHTML = `
                <td class="p-3">${item.entity}</td>
                <td class="p-3">${item.capabilities}</td>
                <td class="p-3">${item.obstacles}</td>
                <td class="p-3">${item.needs}</td>
                <td class="p-3 flex space-x-reverse space-x-2">
                    <button class="view-item-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-type="performanceAnalysis" data-id="${item.id}">عرض</button>
                    <button class="edit-item-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-type="performanceAnalysis" data-id="${item.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="performanceAnalysis" data-id="${item.id}">حذف</button>
                </td>
            `;
            tableBody.appendChild(row);
        });
        // Attach event listeners using delegation
        tableBody.querySelectorAll('.view-item-btn[data-type="performanceAnalysis"]').forEach(btn => btn.addEventListener('click', function() { viewPerformanceAnalysisDetails(parseInt(this.dataset.id)); }));
        tableBody.querySelectorAll('.edit-item-btn[data-type="performanceAnalysis"]').forEach(btn => btn.addEventListener('click', function() { openPerformanceAnalysisForm(performanceAnalysisData.find(pa => pa.id === parseInt(this.dataset.id))); }));
    }

    document.getElementById('addPerformanceAnalysisBtn').addEventListener('click', () => openPerformanceAnalysisForm());

    function openPerformanceAnalysisForm(item = null) {
        genericFormTitle.textContent = item ? 'تعديل تقرير تحليل الأداء' : 'إضافة تقرير تحليل أداء جديد';
        modalFormFields.innerHTML = `
            <div>
                <label for="paEntity" class="block text-sm font-medium text-gray-700">المديرية/الشعبة:</label>
                <input type="text" id="paEntity" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div class="md:col-span-2">
                <label for="paCapabilities" class="block text-sm font-medium text-gray-700">أبرز الإمكانات:</label>
                <textarea id="paCapabilities" rows="2" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required></textarea>
            </div>
            <div class="md:col-span-2">
                <label for="paObstacles" class="block text-sm font-medium text-gray-700">التحديات/العقبات:</label>
                <textarea id="paObstacles" rows="2" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required></textarea>
            </div>
            <div class="md:col-span-2">
                <label for="paNeeds" class="block text-sm font-medium text-gray-700">الاحتياجات الرئيسية:</label>
                <textarea id="paNeeds" rows="2" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required></textarea>
            </div>
        `;
        if (item) {
            document.getElementById('paEntity').value = item.entity;
            document.getElementById('paCapabilities').value = item.capabilities;
            document.getElementById('paObstacles').value = item.obstacles;
            document.getElementById('paNeeds').value = item.needs;
            currentPerformanceAnalysisId = item.id;
        } else {
            genericForm.reset();
            currentPerformanceAnalysisId = null;
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); savePerformanceAnalysis(); };
    }

    function savePerformanceAnalysis() {
        const entity = document.getElementById('paEntity').value;
        const capabilities = document.getElementById('paCapabilities').value;
        const obstacles = document.getElementById('paObstacles').value;
        const needs = document.getElementById('paNeeds').value;

        if (!entity || !capabilities || !obstacles || !needs) {
            console.error('يرجى ملء جميع الحقول المطلوبة لتقرير تحليل الأداء.');
            return;
        }

        if (currentPerformanceAnalysisId) {
            const index = performanceAnalysisData.findIndex(pa => pa.id === currentPerformanceAnalysisId);
            if (index !== -1) {
                performanceAnalysisData[index] = { ...performanceAnalysisData[index], entity, capabilities, obstacles, needs };
            }
        } else {
            const newId = performanceAnalysisData.length > 0 ? Math.max(...performanceAnalysisData.map(pa => pa.id)) + 1 : 1;
            performanceAnalysisData.push({ id: newId, entity, capabilities, obstacles, needs });
        }
        saveData('performanceAnalysisData', performanceAnalysisData); // Save to localStorage
        hideGenericFormModal();
        populatePerformanceAnalysisTable();
    }

    function deletePerformanceAnalysis(id) {
        performanceAnalysisData = performanceAnalysisData.filter(pa => pa.id !== id);
        saveData('performanceAnalysisData', performanceAnalysisData); // Save to localStorage
        populatePerformanceAnalysisTable();
    }

    function viewPerformanceAnalysisDetails(id) {
        const item = performanceAnalysisData.find(pa => pa.id === id);
        if (!item) return;
        detailModalTitle.textContent = `تفاصيل تحليل الأداء: ${item.entity}`;
        detailModalContent.innerHTML = `
            <p><strong class="text-sky-700">المديرية/الشعبة:</strong> ${item.entity}</p>
            <p><strong class="text-sky-700">أبرز الإمكانات:</strong> ${item.capabilities}</p>
            <p><strong class="text-sky-700">التحديات/العقبات:</strong> ${item.obstacles}</p>
            <p><strong class="text-sky-700">الاحتياجات الرئيسية:</strong> ${item.needs}</p>
        `;
        genericDetailModal.classList.remove('hidden');
        detailModalBackBtn.dataset.returnTo = 'reports'; // Set return view
    }

    // --- Directorates View Functions ---
    function populateDirectoratesView(data = directoratesData) {
        const directoratesGrid = document.getElementById('directoratesGrid');
        directoratesGrid.innerHTML = ''; // Clear existing content

        if (data.length === 0) {
            directoratesGrid.innerHTML = `
                <div class="col-span-full text-center text-gray-500 p-8">
                    <p class="text-lg mb-4">لا توجد بيانات مديريات حالياً.</p>
                    <p>يمكنك إضافة مديرية جديدة باستخدام زر "إضافة مديرية جديدة".</p>
                </div>
            `;
            return;
        }

        data.forEach(dir => {
            const card = document.createElement('div');
            card.className = 'bg-white p-6 rounded-xl shadow-md border-t-4 border-blue-500 flex flex-col';
            card.innerHTML = `
                <h3 class="font-bold text-lg text-blue-700 mb-2">${dir.name}</h3>
                <p class="text-sm text-gray-600 mb-4">${dir.description}</p>
                <div class="flex-grow">
                    <p class="text-gray-700"><strong class="text-blue-600">الموقع:</strong> ${dir.location}</p>
                    <p class="text-gray-700"><strong class="text-blue-600">عدد الأقسام:</strong> ${dir.sections}</p>
                    <p class="text-gray-700"><strong class="text-blue-600">عدد الموظفين:</strong> ${dir.employees}</p>
                    <!-- Removed "عدد الأوقاف" from display -->
                </div>
                <div class="flex justify-end space-x-reverse space-x-2 mt-4">
                    <button class="manage-departments-btn bg-sky-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-sky-600 transition-colors" data-id="${dir.id}">إدارة الشعب</button>
                    <button class="edit-directorate-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-id="${dir.id}">تعديل</button>
                    <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="directorate" data-id="${dir.id}">حذف</button>
                </div>
            `;
            directoratesGrid.appendChild(card);
        });

        // Add event listeners for "Edit" buttons
        directoratesGrid.querySelectorAll('.edit-directorate-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                openDirectorateForm(directoratesData.find(d => d.id === parseInt(this.dataset.id)));
            });
        });

        // Add event listeners for "Manage Departments" buttons
        directoratesGrid.querySelectorAll('.manage-departments-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                openDepartmentManagementModal(parseInt(this.dataset.id));
            });
        });
    }

    // Add button for Directorates
    document.getElementById('addDirectorateBtn').addEventListener('click', () => openDirectorateForm());

    function openDirectorateForm(directorate = null) {
        genericFormTitle.textContent = directorate ? 'تعديل بيانات المديرية' : 'إضافة مديرية جديدة';
        modalFormFields.innerHTML = `
            <div>
                <label for="dirName" class="block text-sm font-medium text-gray-700">اسم المديرية:</label>
                <input type="text" id="dirName" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="dirLocation" class="block text-sm font-medium text-gray-700">الموقع:</label>
                <input type="text" id="dirLocation" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="dirSections" class="block text-sm font-medium text-gray-700">عدد الأقسام:</label>
                <input type="number" id="dirSections" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="dirEmployees" class="block text-sm font-medium text-gray-700">عدد الموظفين:</label>
                <input type="number" id="dirEmployees" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div class="md:col-span-2">
                <label for="dirDescription" class="block text-sm font-medium text-gray-700">الوصف:</label>
                <textarea id="dirDescription" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
        `;
        if (directorate) {
            document.getElementById('dirName').value = directorate.name;
            document.getElementById('dirLocation').value = directorate.location;
            document.getElementById('dirSections').value = directorate.sections;
            document.getElementById('dirEmployees').value = directorate.employees;
            document.getElementById('dirDescription').value = directorate.description || '';
            currentDirectorateId = directorate.id;
        } else {
            genericForm.reset();
            currentDirectorateId = null;
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); saveDirectorate(); };
    }

    function saveDirectorate() {
        const name = document.getElementById('dirName').value;
        const location = document.getElementById('dirLocation').value;
        const sections = parseInt(document.getElementById('dirSections').value);
        const employees = parseInt(document.getElementById('dirEmployees').value);
        const description = document.getElementById('dirDescription').value;

        if (!name || !location || isNaN(sections) || isNaN(employees)) {
            console.error('يرجى ملء جميع الحقول المطلوبة للمديرية بالأرقام الصحيحة.');
            return;
        }

        if (currentDirectorateId) {
            const index = directoratesData.findIndex(d => d.id === currentDirectorateId);
            if (index !== -1) {
                // Preserve existing departments if updating
                directoratesData[index] = { ...directoratesData[index], name, location, sections, employees, description };
            }
        } else {
            const newId = directoratesData.length > 0 ? Math.max(...directoratesData.map(d => d.id)) + 1 : 1;
            // Initialize departments array for new directorate
            directoratesData.push({ id: newId, name, location, sections, employees, description, departments: [] });
        }
        saveData('directoratesData', directoratesData); // Save to localStorage
        hideGenericFormModal();
        populateDirectoratesView();
    }

    function deleteDirectorate(id) {
        directoratesData = directoratesData.filter(d => d.id !== id);
        saveData('directoratesData', directoratesData); // Save to localStorage
        populateDirectoratesView();
    }

    // --- Department Management Functions ---
    // This function now replaces viewDirectorateDetails and handles department listing
    function openDepartmentManagementModal(directorateId) {
        currentDirectorateId = directorateId; // Set the global current directorate ID
        const directorate = directoratesData.find(d => d.id === directorateId);
        if (!directorate) return;

        detailModalTitle.textContent = `إدارة شعب المديرية: ${directorate.name}`;
        
        // Build the departments table
        let departmentsTableHTML = `
            <div class="flex justify-end mb-4">
                <button id="addDepartmentBtn" class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors">إضافة شعبة جديدة</button>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-right border">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="p-3 sortable" data-sort-by="name">اسم الشعبة</th>
                            <th class="p-3 sortable" data-sort-by="head">المسؤول</th>
                            <th class="p-3 sortable" data-sort-by="employees">عدد الموظفين</th>
                            <th class="p-3 sortable" data-sort-by="publicSchools">مدارس عامة</th>
                            <th class="p-3 sortable" data-sort-by="religiousSchools">مدارس دينية</th>
                            <th class="p-3 sortable" data-sort-by="mosques">مساجد</th>
                            <th class="p-3 sortable" data-sort-by="hussainiyas">حسينيات</th>
                            <th class="p-3">الإجراءات</th>
                        </tr>
                    </thead>
                    <tbody id="departmentTableBody">
        `;
        // Use a temporary variable for departments data to allow sorting without modifying original
        const departmentsToDisplay = directorate.departments || [];

        if (departmentsToDisplay.length > 0) {
            departmentsToDisplay.forEach(dept => {
                departmentsTableHTML += `
                    <tr class="border-b hover:bg-gray-50">
                        <td class="p-3">${dept.name}</td>
                        <td class="p-3">${dept.head || 'غير محدد'}</td>
                        <td class="p-3">${dept.employees || 0}</td>
                        <td class="p-3">${dept.publicSchools !== undefined ? dept.publicSchools : 0}</td>
                        <td class="p-3">${dept.religiousSchools !== undefined ? dept.religiousSchools : 0}</td>
                        <td class="p-3">${dept.mosques !== undefined ? dept.mosques : 0}</td>
                        <td class="p-3">${dept.hussainiyas !== undefined ? dept.hussainiyas : 0}</td>
                        <td class="p-3 flex space-x-reverse space-x-2">
                            <button class="edit-department-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-id="${dept.id}">تعديل</button>
                            <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="department" data-directorate-id="${directorate.id}" data-id="${dept.id}">حذف</button>
                        </td>
                    </tr>
                `;
            });
        } else {
            departmentsTableHTML += `
                <tr>
                    <td colspan="8" class="p-3 text-center text-gray-500">لا توجد شعب مسجلة لهذه المديرية.</td>
                </tr>
            `;
        }
        departmentsTableHTML += `
                    </tbody>
                </table>
            </div>
            <p class="mt-4 text-gray-600">هنا يمكن إضافة المزيد من التفاصيل الخاصة بالمديرية مثل المشاريع، التحديات، والإنجازات.</p>
        `;
        detailModalContent.innerHTML = departmentsTableHTML;

        // Attach event listeners for department buttons
        document.getElementById('addDepartmentBtn').addEventListener('click', () => openDepartmentForm(directorateId));
        detailModalContent.querySelectorAll('.edit-department-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                const deptId = parseInt(this.dataset.id);
                const dept = directorate.departments.find(d => d.id === deptId);
                openDepartmentForm(directorateId, dept);
            });
        });
        
        // Setup sorting for departments table
        setupTableSorting('departmentTableBody', directorate.departments || [], (sortedDepts) => {
            // Re-render the departments table with sorted data
            const tableBody = document.getElementById('departmentTableBody');
            tableBody.innerHTML = '';
            if (sortedDepts && sortedDepts.length > 0) {
                sortedDepts.forEach(dept => {
                    const row = document.createElement('tr');
                    row.className = 'border-b hover:bg-gray-50';
                    row.innerHTML = `
                        <td class="p-3">${dept.name}</td>
                        <td class="p-3">${dept.head || 'غير محدد'}</td>
                        <td class="p-3">${dept.employees || 0}</td>
                        <td class="p-3">${dept.publicSchools !== undefined ? dept.publicSchools : 0}</td>
                        <td class="p-3">${dept.religiousSchools !== undefined ? dept.religiousSchools : 0}</td>
                        <td class="p-3">${dept.mosques !== undefined ? dept.mosques : 0}</td>
                        <td class="p-3">${dept.hussainiyas !== undefined ? dept.hussainiyas : 0}</td>
                        <td class="p-3 flex space-x-reverse space-x-2">
                            <button class="edit-department-btn bg-amber-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-amber-600 transition-colors" data-id="${dept.id}">تعديل</button>
                            <button class="delete-item-btn bg-red-500 text-white px-3 py-1 rounded-lg text-sm hover:bg-red-600 transition-colors" data-type="department" data-directorate-id="${directorate.id}" data-id="${dept.id}">حذف</button>
                        </td>
                    `;
                    tableBody.appendChild(row);
                });
                // Re-attach event listeners after re-rendering
                tableBody.querySelectorAll('.edit-department-btn').forEach(btn => {
                    btn.addEventListener('click', function() {
                        const deptId = parseInt(this.dataset.id);
                        const dept = directorate.departments.find(d => d.id === deptId);
                        openDepartmentForm(directorateId, dept);
                    });
                });
            } else {
                 tableBody.innerHTML = `
                    <tr>
                        <td colspan="8" class="p-3 text-center text-gray-500">لا توجد شعب مسجلة لهذه المديرية.</td>
                    </tr>
                `;
            }
        }, 'department');


        genericDetailModal.classList.remove('hidden');
        detailModalBackBtn.dataset.returnTo = 'directorates'; // Ensure back button returns to directorates view
    }

    function openDepartmentForm(directorateId, department = null) {
        currentDirectorateId = directorateId; // Ensure directorateId is set for saving
        currentDepartmentId = department ? department.id : null; // Set department ID for editing

        genericFormTitle.textContent = department ? 'تعديل بيانات الشعبة' : 'إضافة شعبة جديدة';
        modalFormFields.innerHTML = `
            <div>
                <label for="deptName" class="block text-sm font-medium text-gray-700">اسم الشعبة:</label>
                <input type="text" id="deptName" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500" required>
            </div>
            <div>
                <label for="deptHead" class="block text-sm font-medium text-gray-700">المسؤول:</label>
                <input type="text" id="deptHead" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="deptEmployees" class="block text-sm font-medium text-gray-700">عدد الموظفين:</label>
                <input type="number" id="deptEmployees" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="deptPublicSchools" class="block text-sm font-medium text-gray-700">عدد مدارس التعليم العام:</label>
                <input type="number" id="deptPublicSchools" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="deptReligiousSchools" class="block text-sm font-medium text-gray-700">عدد مدارس التعليم الديني:</label>
                <input type="number" id="deptReligiousSchools" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="deptMosques" class="block text-sm font-medium text-gray-700">عدد المساجد:</label>
                <input type="number" id="deptMosques" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div>
                <label for="deptHussainiyas" class="block text-sm font-medium text-gray-700">عدد الحسينيات:</label>
                <input type="number" id="deptHussainiyas" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500">
            </div>
            <div class="md:col-span-2">
                <label for="deptTasks" class="block text-sm font-medium text-gray-700">المهام الرئيسية:</label>
                <textarea id="deptTasks" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
            <div class="md:col-span-2">
                <label for="deptNeeds" class="block text-sm font-medium text-gray-700">الاحتياجات:</label>
                <textarea id="deptNeeds" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-sky-500 focus:ring-sky-500"></textarea>
            </div>
        `;
        if (department) {
            document.getElementById('deptName').value = department.name;
            document.getElementById('deptHead').value = department.head || '';
            document.getElementById('deptEmployees').value = department.employees || 0;
            document.getElementById('deptPublicSchools').value = department.publicSchools || 0;
            document.getElementById('deptReligiousSchools').value = department.religiousSchools || 0;
            document.getElementById('deptMosques').value = department.mosques || 0;
            document.getElementById('deptHussainiyas').value = department.hussainiyas || 0;
            document.getElementById('deptTasks').value = department.tasks || '';
            document.getElementById('deptNeeds').value = department.needs || '';
        } else {
            genericForm.reset();
        }
        genericFormModal.classList.remove('hidden');
        genericForm.onsubmit = (e) => { e.preventDefault(); saveDepartment(); };
    }

    function saveDepartment() {
        const directorate = directoratesData.find(d => d.id === currentDirectorateId);
        if (!directorate) {
            console.error('المديرية غير موجودة.');
            return;
        }

        const name = document.getElementById('deptName').value;
        const head = document.getElementById('deptHead').value;
        const employees = parseInt(document.getElementById('deptEmployees').value) || 0;
        const publicSchools = parseInt(document.getElementById('deptPublicSchools').value) || 0;
        const religiousSchools = parseInt(document.getElementById('deptReligiousSchools').value) || 0;
        const mosques = parseInt(document.getElementById('deptMosques').value) || 0;
        const hussainiyas = parseInt(document.getElementById('deptHussainiyas').value) || 0;
        const tasks = document.getElementById('deptTasks').value;
        const needs = document.getElementById('deptNeeds').value;


        if (!name) {
            console.error('يرجى ملء اسم الشعبة.');
            return;
        }

        if (currentDepartmentId) {
            const index = directorate.departments.findIndex(d => d.id === currentDepartmentId);
            if (index !== -1) {
                directorate.departments[index] = { ...directorate.departments[index], name, head, employees, publicSchools, religiousSchools, mosques, hussainiyas, tasks, needs };
            }
        } else {
            const newId = directorate.departments.length > 0 ? Math.max(...directorate.departments.map(d => d.id)) + 1 : 1;
            directorate.departments.push({ id: newId, name, head, employees, publicSchools, religiousSchools, mosques, hussainiyas, tasks, needs });
        }
        saveData('directoratesData', directoratesData); // Save updated directorates data
        hideGenericFormModal();
        openDepartmentManagementModal(currentDirectorateId); // Re-open department management modal to show updates
    }

    function deleteDepartment(directorateId, departmentId) {
        const directorate = directoratesData.find(d => d.id === directorateId);
        if (!directorate) {
            console.error('المديرية غير موجودة.');
            return;
        }
        directorate.departments = directorate.departments.filter(d => d.id !== departmentId);
        saveData('directoratesData', directoratesData); // Save updated directorates data
        openDepartmentManagementModal(directorateId); // Re-open department management modal to show updates
    }


    // --- Organizational Chart Node Click Handler ---
    document.querySelectorAll('.org-chart .node').forEach(node => {
        node.addEventListener('click', function() {
            const orgUnitId = this.dataset.orgUnitId;
            const unitDetails = orgUnitDetails[orgUnitId];

            if (unitDetails) {
                detailModalTitle.textContent = `تفاصيل: ${unitDetails.name}`;
                let responsibilitiesHtml = '';
                if (unitDetails.responsibilities && unitDetails.responsibilities.length > 0) {
                    responsibilitiesHtml = unitDetails.responsibilities.map(resp => `<li>${resp}</li>`).join('');
                } else {
                    responsibilitiesHtml = '<li>لا توجد مسؤوليات محددة.</li>';
                }

                detailModalContent.innerHTML = `
                    <p><strong class="text-sky-700">الوحدة:</strong> ${unitDetails.name}</p>
                    <p><strong class="text-sky-700">الوصف:</strong> ${unitDetails.description || 'لا يوجد وصف.'}</p>
                    <p><strong class="text-sky-700">المسؤول:</strong> ${unitDetails.head || 'غير محدد'}</p>
                    <p class="mt-2"><strong class="text-sky-700">المهام/المسؤوليات الرئيسية:</strong></p>
                    <ul class="list-disc list-inside">${responsibilitiesHtml}</ul>
                `;
                genericDetailModal.classList.remove('hidden');
                detailModalBackBtn.dataset.returnTo = 'org-structure'; // Set return view
            } else {
                console.warn(`No details found for organizational unit: ${orgUnitId}`);
            }
        });
    });


    // --- Generic Modal Control Functions ---
    closeGenericFormModalBtn.addEventListener('click', hideGenericFormModal);
    cancelGenericFormBtn.addEventListener('click', hideGenericFormModal);
    closeGenericDetailModalBtn.addEventListener('click', hideGenericDetailModal);
    detailModalBackBtn.addEventListener('click', () => {
        hideGenericDetailModal();
        // Determine where to go back based on context
        if (detailModalBackBtn.dataset.returnTo === 'reports') {
            document.getElementById('reports-view').classList.remove('hidden');
        } else if (detailModalBackBtn.dataset.returnTo === 'hr') {
            showView('hr');
        } else if (detailModalBackBtn.dataset.returnTo === 'operations') {
            showView('operations');
        } else if (detailModalBackBtn.dataset.returnTo === 'org-structure') {
            showView('org-structure');
        } else if (detailModalBackBtn.dataset.returnTo === 'directorates') {
            showView('directorates');
        }
        // If no specific returnTo is set, stay on current primary view.
    });

    function hideGenericFormModal() {
        genericFormModal.classList.add('hidden');
        genericForm.reset();
        modalFormFields.innerHTML = '';
    }

    function hideGenericDetailModal() {
        genericDetailModal.classList.add('hidden');
        detailModalContent.innerHTML = '';
        detailModalBackBtn.dataset.returnTo = ''; // Clear return context
    }

    function hideConfirmationModal() {
        confirmationModal.classList.add('hidden');
        itemToDelete = { type: null, id: null, callback: null, directorateId: null }; // Reset directorateId as well
    }

    // --- Centralized Delete Logic ---
    document.body.addEventListener('click', function(e) {
        if (e.target.classList.contains('delete-item-btn')) {
            const type = e.target.dataset.type;
            const id = parseInt(e.target.dataset.id);
            const directorateId = e.target.dataset.directorateId ? parseInt(e.target.dataset.directorateId) : null; // Get directorateId if present
            let callback;

            switch (type) {
                case 'employee': callback = deleteEmployee; break;
                case 'correspondence': callback = deleteCorrespondence; break;
                case 'announcement': callback = deleteAnnouncement; break;
                case 'activity': callback = deleteActivity; break;
                case 'visitReport': callback = deleteVisitReport; break;
                case 'performanceAnalysis': callback = deletePerformanceAnalysis; break; 
                case 'directorate': callback = deleteDirectorate; break;
                case 'department': callback = deleteDepartment; break; // New case for department
                default: return;
            }
            itemToDelete = { type, id, callback, directorateId }; // Store directorateId
            confirmationModal.classList.remove('hidden');
        } 
    });

    closeModalBtn.addEventListener('click', hideConfirmationModal);
    cancelDeleteBtn.addEventListener('click', hideConfirmationModal);
    confirmDeleteBtn.addEventListener('click', () => {
        if (itemToDelete.type && itemToDelete.id !== null && itemToDelete.callback) {
            if (itemToDelete.type === 'department') {
                itemToDelete.callback(itemToDelete.directorateId, itemToDelete.id); // Pass both IDs for department deletion
            } else {
                itemToDelete.callback(itemToDelete.id);
            }
        }
        hideConfirmationModal();
    });

    // --- Chart.js Initialization ---
    const createDonutChart = (ctx, label, data, colors) => {
        new Chart(ctx, {
            type: 'doughnut',
            data: {
                labels: [label, ''],
                datasets: [{
                    data: data,
                    backgroundColor: colors,
                    borderColor: 'rgba(0,0,0,0)',
                    borderWidth: 1,
                    cutout: '80%'
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { display: false },
                    tooltip: { enabled: false }
                }
            }
        });
    };
    
    createDonutChart(document.getElementById('financialChart').getContext('2d'), 'هدف مالي', [115, 100-115], ['#22c55e', '#e5e7eb']);
    createDonutChart(document.getElementById('customerChart').getContext('2d'), 'رضا المستفيدين', [92, 8], ['#0ea5e9', '#e5e7eb']);
    createDonutChart(document.getElementById('processChart').getContext('2d'), 'أتمتة العمليات', [85, 15], ['#f59e0b', '#e5e7eb']);
    createDonutChart(document.getElementById('learningChart').getContext('2d'), 'موظفين مدربين', [75, 25], ['#a855f7', '#e5e7eb']);

    // Initial load functions and setup sorting
    showView('dashboard'); // Start with dashboard view
    
    populateEmployeeTable();
    setupTableSorting('employeeTableBody', employeeData, populateEmployeeTable, 'employee');

    populateCorrespondenceTable('incoming');
    setupTableSorting('incomingCorrespondenceTableBody', correspondenceData.filter(m => m.type === 'وارد'), (data) => populateCorrespondenceTable('incoming', incomingSearchInput.value, data), 'incomingCorrespondence');
    setupTableSorting('outgoingCorrespondenceTableBody', correspondenceData.filter(m => m.type === 'صادر'), (data) => populateCorrespondenceTable('outgoing', outgoingSearchInput.value, data), 'outgoingCorrespondence');
    
    populateAnnouncementTable();
    setupTableSorting('announcementTableBody', announcementData, populateAnnouncementTable, 'announcement');

    populateActivityTable();
    setupTableSorting('activityTableBody', activityData, populateActivityTable, 'activity');

    populateVisitReportTable();
    setupTableSorting('visitReportTableBody', visitReportData, populateVisitReportTable, 'visitReport');

    populatePerformanceAnalysisTable(); 
    setupTableSorting('performanceAnalysisTableBody', performanceAnalysisData, populatePerformanceAnalysisTable, 'performanceAnalysis');

    // Populate the new directorates view
    populateDirectoratesView();

    // Event listener for the "Back to HR" button
    document.getElementById('back-to-hr').addEventListener('click', () => {
        console.log('Back to HR button clicked. Navigating to HR view.'); // Debug log
        showView('hr'); // Call showView to switch to the HR view
    });
});
</script>

</body>
</html>
# read
