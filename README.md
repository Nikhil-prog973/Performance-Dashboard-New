<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Team Performance Dashboard</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.0/dist/chart.min.js"></script>
    <!-- Inter Font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            color: #374151;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 2rem;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 1.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border-radius: 0.5rem;
            overflow: hidden; /* Ensures rounded corners apply to content */
        }
        th, td {
            padding: 1rem;
            text-align: left;
            border-bottom: 1px solid #e5e7eb;
        }
        th {
            background-color: #1f2937;
            color: #f9fafb;
            font-weight: 600;
            text-transform: uppercase;
            font-size: 0.875rem;
        }
        /* No specific background for tr:nth-child(even) or tr:hover here,
           as we will apply dynamic background colors based on pending items. */
        tr:hover {
            opacity: 0.9; /* Slight hover effect on dynamically colored rows */
        }
        .chart-container {
            position: relative;
            height: 400px; /* Fixed height for charts */
            width: 100%;
            margin-top: 2rem;
            background-color: #ffffff;
            padding: 1.5rem;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .file-input-wrapper {
            display: flex;
            align-items: center;
            gap: 1rem;
        }
        .file-input-label {
            background-color: #4f46e5;
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            cursor: pointer;
            transition: background-color 0.2s ease-in-out;
            font-weight: 500;
        }
        .file-input-label:hover {
            background-color: #4338ca;
        }
        input[type="file"] {
            display: none;
        }
        .message-box {
            background-color: #dbeafe;
            color: #1e40af;
            padding: 1rem;
            border-radius: 0.5rem;
            margin-top: 1.5rem;
            border: 1px solid #93c5fd;
        }
        .llm-button {
            background-color: #10b981; /* Green */
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            cursor: pointer;
            transition: background-color 0.2s ease-in-out;
            font-weight: 500;
            margin-top: 1rem;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 0.5rem;
        }
        .llm-button:hover {
            background-color: #059669; /* Darker Green */
        }
        .llm-button:disabled {
            background-color: #a7f3d0; /* Lighter green for disabled */
            cursor: not-allowed;
        }
        .loading-spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid #ffffff;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .insights-section {
            background-color: #ffffff;
            padding: 1.5rem;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            margin-top: 2rem;
        }
        .insights-section h2 {
            margin-bottom: 1rem;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-4xl font-bold text-center mb-8 text-gray-800">Team Performance Dashboard</h1>

        <div class="bg-white p-6 rounded-lg shadow-md mb-8">
            <h2 class="text-2xl font-semibold mb-4 text-gray-700">Upload Performance Data (CSV)</h2>
            <p class="text-gray-600 mb-4">
                Please export your Google Sheet as a CSV file (File > Download > Comma Separated Values (.csv)) and upload it here.
            </p>
            <div class="file-input-wrapper">
                <label for="csvFileInput" class="file-input-label">
                    Choose CSV File
                </label>
                <input type="file" id="csvFileInput" accept=".csv">
                <span id="fileNameDisplay" class="text-gray-500 italic">No file chosen</span>
            </div>
            <div id="messageBox" class="message-box hidden mt-4"></div>
        </div>

        <div class="bg-white p-6 rounded-lg shadow-md mb-8">
            <h2 class="text-2xl font-semibold mb-4 text-gray-700">Team Performance Overview</h2>
            <div class="overflow-x-auto rounded-lg">
                <table id="performanceTable">
                    <thead>
                        <tr>
                            <th>Employee Name</th>
                            <th>Monthly Target</th>
                            <th>Pending Items</th>
                        </tr>
                    </thead>
                    <tbody>
                        <!-- Data will be inserted here by JavaScript -->
                    </tbody>
                </table>
            </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
            <div class="chart-container">
                <h2 class="text-xl font-semibold mb-4 text-gray-700">Pending Items per Agent</h2>
                <canvas id="pendingItemsChart"></canvas>
            </div>
            <div class="chart-container">
                <h2 class="text-xl font-semibold mb-4 text-gray-700">Target vs. Pending Comparison</h2>
                <canvas id="targetVsPendingChart"></canvas>
            </div>
        </div>

        <div class="insights-section">
            <h2 class="text-2xl font-semibold text-gray-700">Performance Insights ✨</h2>
            <button id="generateInsightsBtn" class="llm-button" disabled>
                Generate Performance Insights ✨
                <span id="insightsLoadingSpinner" class="hidden loading-spinner"></span>
            </button>
            <div id="insightsOutput" class="mt-4 p-4 bg-gray-50 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                Click "Generate Performance Insights ✨" to get an AI-powered summary of your team's data.
            </div>
        </div>
    </div>

    <script>
        // Global chart instances to allow destruction and recreation
        let pendingItemsChartInstance = null;
        let targetVsPendingChartInstance = null;
        let currentPerformanceData = []; // Store parsed data globally

        // Get references to HTML elements
        const csvFileInput = document.getElementById('csvFileInput');
        const fileNameDisplay = document.getElementById('fileNameDisplay');
        const performanceTableBody = document.querySelector('#performanceTable tbody');
        const messageBox = document.getElementById('messageBox');
        const pendingItemsChartCanvas = document.getElementById('pendingItemsChart');
        const targetVsPendingChartCanvas = document.getElementById('targetVsPendingChart');
        const generateInsightsBtn = document.getElementById('generateInsightsBtn');
        const insightsLoadingSpinner = document.getElementById('insightsLoadingSpinner');
        const insightsOutput = document.getElementById('insightsOutput');

        /**
         * Displays a message in the message box.
         * @param {string} message - The message to display.
         * @param {string} type - The type of message ('success', 'error', 'info').
         */
        function showMessage(message, type = 'info') {
            messageBox.textContent = message;
            messageBox.classList.remove('hidden', 'bg-red-100', 'text-red-700', 'border-red-400', 'bg-green-100', 'text-green-700', 'border-green-400', 'bg-blue-100', 'text-blue-700', 'border-blue-400');
            if (type === 'error') {
                messageBox.classList.add('bg-red-100', 'text-red-700', 'border-red-400');
            } else if (type === 'success') {
                messageBox.classList.add('bg-green-100', 'text-green-700', 'border-green-400');
            } else { // info
                messageBox.classList.add('bg-blue-100', 'text-blue-700', 'border-blue-400');
            }
            messageBox.classList.remove('hidden');
        }

        /**
         * Hides the message box.
         */
        function hideMessageBox() {
            messageBox.classList.add('hidden');
        }

        /**
         * Parses CSV data and extracts relevant columns.
         * @param {string} csvText - The raw CSV content.
         * @returns {Array<Object>} An array of objects, each representing an employee's data.
         */
        function parseCSV(csvText) {
            const lines = csvText.split(/\r?\n/).filter(line => line.trim() !== ''); // Split by new line, handle Windows/Unix, filter empty lines
            if (lines.length === 0) {
                showMessage('The uploaded CSV file is empty or unreadable.', 'error');
                return [];
            }

            const data = [];
            // Assuming the first row is a header and skipping it
            for (let i = 1; i < lines.length; i++) {
                const columns = lines[i].split(','); // Simple split by comma. For more robust CSV, a dedicated parser would be better.
                
                // Check if the row has enough columns
                // Need at least 70 columns for BR (index 69)
                if (columns.length > 69) { 
                    const employeeName = columns[3] ? columns[3].trim() : 'N/A'; // Column D (index 3)
                    const monthlyTarget = parseInt(columns[68], 10); // Column BQ (index 68)
                    const pendingItems = parseInt(columns[69], 10); // Column BR (index 69)

                    // Only add valid entries
                    if (!isNaN(monthlyTarget) && !isNaN(pendingItems) && employeeName !== 'N/A' && employeeName !== '') {
                        data.push({
                            employeeName: employeeName,
                            monthlyTarget: monthlyTarget,
                            pendingItems: pendingItems
                        });
                    }
                }
            }
            return data;
        }

        /**
         * Renders the performance data into the HTML table.
         * @param {Array<Object>} data - The processed employee data.
         */
        function renderTable(data) {
            performanceTableBody.innerHTML = ''; // Clear existing table rows
            if (data.length === 0) {
                const row = performanceTableBody.insertRow();
                const cell = row.insertCell();
                cell.colSpan = 3;
                cell.textContent = 'No data available to display.';
                cell.classList.add('text-center', 'py-4', 'text-gray-500');
                return;
            }

            // Sort data by pending items in descending order
            data.sort((a, b) => b.pendingItems - a.pendingItems);

            // Determine min and max pending items for color scaling
            const pendingValues = data.map(item => item.pendingItems);
            const minPending = Math.min(...pendingValues);
            const maxPending = Math.max(...pendingValues);

            data.forEach(item => {
                const row = performanceTableBody.insertRow();
                row.insertCell().textContent = item.employeeName;
                row.insertCell().textContent = item.monthlyTarget;
                row.insertCell().textContent = item.pendingItems;

                // Calculate color shade based on pending items
                // Hue for orange/yellow is around 30-40, Saturation 100%, Lightness from 95% (light) to 70% (darker)
                let lightness = 95; // Default for very low pending
                if (maxPending > minPending) { // Avoid division by zero if all pending items are same
                    // Scale pending items to a 0-1 range, then map to lightness range (e.g., 95 to 70)
                    const normalizedPending = (item.pendingItems - minPending) / (maxPending - minPending);
                    lightness = 95 - (normalizedPending * 25); // 95 - (0-1 * 25) -> 95 to 70
                }
                const backgroundColor = `hsl(35, 100%, ${lightness}%)`; // HSL for orange/yellow shades

                row.style.backgroundColor = backgroundColor;
            });
        }

        /**
         * Renders the Chart.js charts.
         * @param {Array<Object>} data - The processed employee data.
         */
        function renderCharts(data) {
            // Destroy existing chart instances if they exist
            if (pendingItemsChartInstance) {
                pendingItemsChartInstance.destroy();
            }
            if (targetVsPendingChartInstance) {
                targetVsPendingChartInstance.destroy();
            }

            if (data.length === 0) {
                // Optionally show a message on the canvas or hide it
                return;
            }

            const labels = data.map(item => item.employeeName);
            const pendingItems = data.map(item => item.pendingItems);
            const monthlyTargets = data.map(item => item.monthlyTarget);

            // Determine colors for Target vs Pending chart
            const backgroundColors = data.map(item => {
                // If pending items are more than 80% of the target, highlight red/orange
                // If pending items are less than or equal to 80% of the target, highlight green
                if (item.monthlyTarget > 0 && item.pendingItems / item.monthlyTarget > 0.8) {
                    return 'rgba(239, 68, 68, 0.8)'; // Red
                } else if (item.monthlyTarget > 0 && item.pendingItems / item.monthlyTarget > 0.5) {
                    return 'rgba(245, 158, 11, 0.8)'; // Orange
                }
                return 'rgba(16, 185, 129, 0.8)'; // Green
            });

            // Chart 1: Pending Items per Agent
            pendingItemsChartInstance = new Chart(pendingItemsChartCanvas, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Pending Items',
                        data: pendingItems,
                        backgroundColor: 'rgba(79, 70, 229, 0.8)', // Indigo
                        borderColor: 'rgba(79, 70, 229, 1)',
                        borderWidth: 1,
                        borderRadius: 5,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top',
                        },
                        title: {
                            display: false, // Title handled by H2 tag
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Number of Items'
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Employee Name'
                            }
                        }
                    }
                }
            });

            // Chart 2: Target vs Pending Comparison
            targetVsPendingChartInstance = new Chart(targetVsPendingChartCanvas, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            label: 'Monthly Target',
                            data: monthlyTargets,
                            backgroundColor: 'rgba(59, 130, 246, 0.8)', // Blue
                            borderColor: 'rgba(59, 130, 246, 1)',
                            borderWidth: 1,
                            borderRadius: 5,
                        },
                        {
                            label: 'Pending Items',
                            data: pendingItems,
                            backgroundColor: backgroundColors, // Dynamic colors
                            borderColor: backgroundColors.map(color => color.replace('0.8', '1')),
                            borderWidth: 1,
                            borderRadius: 5,
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top',
                        },
                        title: {
                            display: false, // Title handled by H2 tag
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Number of Items'
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Employee Name'
                            }
                        }
                    }
                }
            });
        }

        /**
         * Generates performance insights using the Gemini API.
         */
        async function generatePerformanceInsights() {
            if (currentPerformanceData.length === 0) {
                insightsOutput.textContent = "Please upload CSV data first to generate insights.";
                return;
            }

            generateInsightsBtn.disabled = true;
            insightsLoadingSpinner.classList.remove('hidden');
            insightsOutput.textContent = "Generating insights... Please wait.";

            const performanceSummary = currentPerformanceData.map(item => 
                `Employee: ${item.employeeName}, Target: ${item.monthlyTarget}, Pending: ${item.pendingItems}`
            ).join('\n');

            const prompt = `Analyze the following team performance data and provide a concise summary of insights. Highlight top performers, agents with high pending items, and any noticeable trends or areas of concern. Suggest general actionable advice based on the data.

Team Performance Data:
${performanceSummary}

Please provide the insights in a clear, easy-to-read format.`;

            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });
            const payload = { contents: chatHistory };
            const apiKey = ""; // Leave as-is, Canvas will provide at runtime
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    insightsOutput.textContent = text;
                } else {
                    insightsOutput.textContent = "Failed to generate insights. Unexpected response from API.";
                    console.error("Gemini API response structure unexpected:", result);
                }
            } catch (error) {
                insightsOutput.textContent = "Error generating insights: " + error.message;
                console.error("Error calling Gemini API:", error);
            } finally {
                generateInsightsBtn.disabled = false;
                insightsLoadingSpinner.classList.add('hidden');
            }
        }

        // Event listener for file input change
        csvFileInput.addEventListener('change', (event) => {
            const file = event.target.files[0];
            if (file) {
                fileNameDisplay.textContent = file.name;
                hideMessageBox(); // Hide previous messages

                const reader = new FileReader();
                reader.onload = (e) => {
                    try {
                        const csvText = e.target.result;
                        currentPerformanceData = parseCSV(csvText); // Store parsed data

                        if (currentPerformanceData.length === 0) {
                            showMessage('No valid data found in the CSV. Please ensure columns D, BQ, and BR contain valid employee names, monthly targets, and pending items respectively, and that the file is not empty.', 'error');
                            renderTable([]); // Clear table
                            renderCharts([]); // Clear charts
                            generateInsightsBtn.disabled = true; // Disable insights button
                            insightsOutput.textContent = "Upload CSV data to generate insights.";
                            return;
                        }

                        renderTable(currentPerformanceData);
                        renderCharts(currentPerformanceData);
                        showMessage('CSV data loaded successfully!', 'success');
                        generateInsightsBtn.disabled = false; // Enable insights button
                        insightsOutput.textContent = "Click \"Generate Performance Insights ✨\" to get an AI-powered summary of your team's data.";
                    } catch (error) {
                        console.error('Error processing CSV:', error);
                        showMessage('Failed to process CSV file. Please ensure it is correctly formatted.', 'error');
                        renderTable([]); // Clear table
                        renderCharts([]); // Clear charts
                        generateInsightsBtn.disabled = true; // Disable insights button
                        insightsOutput.textContent = "Upload CSV data to generate insights.";
                    }
                };
                reader.onerror = () => {
                    showMessage('Error reading file. Please try again.', 'error');
                    renderTable([]); // Clear table
                    renderCharts([]); // Clear charts
                    generateInsightsBtn.disabled = true; // Disable insights button
                    insightsOutput.textContent = "Upload CSV data to generate insights.";
                };
                reader.readAsText(file);
            } else {
                fileNameDisplay.textContent = 'No file chosen';
                showMessage('Please select a CSV file to upload.', 'info');
                renderTable([]); // Clear table
                renderCharts([]); // Clear charts
                generateInsightsBtn.disabled = true; // Disable insights button
                insightsOutput.textContent = "Upload CSV data to generate insights.";
            }
        });

        // Event listener for the new insights button
        generateInsightsBtn.addEventListener('click', generatePerformanceInsights);

        // Initial state: show info message and clear table/charts
        window.onload = () => {
            showMessage('Upload a CSV file to view your team\'s performance data.', 'info');
            renderTable([]);
            renderCharts([]);
            generateInsightsBtn.disabled = true; // Disable insights button initially
        };

    </script>
</body>
</html>
