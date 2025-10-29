<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Step-by-Step Long Division</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for monospaced output and clean lines */
        .steps-container {
            font-family: 'Inter', 'Courier New', Courier, monospace;
            white-space: pre-wrap;
            line-height: 1.6;
        }
        .step-line {
            padding: 4px 0;
            border-bottom: 1px solid #e5e7eb; /* Light gray border */
        }
        .step-final {
            font-weight: bold;
            color: #10b981; /* Green */
        }
        /* Style for the division table setup */
        .division-setup {
            border-left: 3px solid #3b82f6; /* Blue border for the vertical line */
            border-top: 3px solid #3b82f6; /* Blue border for the overhead line */
            padding-left: 0.5rem;
            display: inline-block;
            font-size: 1.5rem;
            line-height: 1.2;
            padding-top: 0.25rem;
        }
    </style>
    <script>
        // Set up Tailwind configuration to use the Inter font
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                },
            },
        }
    </script>
</head>
<body class="bg-gray-50 min-h-screen p-4 font-sans flex items-start justify-center">

    <div class="w-full max-w-lg bg-white shadow-xl rounded-xl p-6 md:p-8 mt-10">
        <h1 class="text-3xl font-extrabold text-blue-600 mb-4 text-center">
            Long Division Demo
        </h1>
        <p class="text-gray-500 mb-6 text-center">Enter a Dividend (up to 9 digits) and a Divisor to see the steps.</p>

        <!-- Input Section -->
        <div class="flex flex-col md:flex-row gap-4 mb-6">
            <input type="number" id="dividend" placeholder="Dividend (e.g., 587)" value="587"
                   class="w-full p-3 border-2 border-blue-200 rounded-lg focus:ring-blue-500 focus:border-blue-500 transition duration-150 shadow-inner text-lg">
            <input type="number" id="divisor" placeholder="Divisor (e.g., 25)" value="25"
                   class="w-full p-3 border-2 border-blue-200 rounded-lg focus:ring-blue-500 focus:border-blue-500 transition duration-150 shadow-inner text-lg">
        </div>
        <button onclick="calculateLongDivision()"
                class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 rounded-lg shadow-lg transition duration-300 ease-in-out transform hover:scale-[1.01] active:scale-[0.99] focus:outline-none focus:ring-4 focus:ring-blue-300">
            Show Division Steps
        </button>

        <!-- Results and Steps Section -->
        <div id="result-container" class="mt-8">
            <div id="final-result" class="text-2xl font-bold text-center mb-4 hidden"></div>
            <div id="steps-output" class="steps-container bg-gray-50 p-4 rounded-lg border border-gray-200 overflow-x-auto min-h-[100px] text-sm">
                <!-- Steps will be injected here -->
                <p class="text-gray-400 text-center">Click 'Show Division Steps' to begin.</p>
            </div>
        </div>

        <!-- Custom Message Box for Errors/Info (instead of alert) -->
        <div id="message-box" class="fixed inset-0 bg-gray-600 bg-opacity-50 hidden items-center justify-center p-4">
            <div class="bg-white rounded-lg p-6 shadow-2xl max-w-sm w-full">
                <h3 class="text-xl font-semibold text-red-600 mb-3">Error</h3>
                <p id="message-content" class="text-gray-700 mb-4"></p>
                <button onclick="document.getElementById('message-box').classList.add('hidden')"
                        class="w-full bg-red-500 hover:bg-red-600 text-white font-bold py-2 rounded-lg transition">
                    Got it
                </button>
            </div>
        </div>
    </div>

    <script>
        /**
         * Shows a custom message box (instead of using alert()).
         * @param {string} message The message content.
         */
        function showMessage(message) {
            document.getElementById('message-content').textContent = message;
            document.getElementById('message-box').classList.remove('hidden');
            document.getElementById('message-box').classList.add('flex');
        }

        /**
         * Calculates and displays the long division process step-by-step.
         */
        function calculateLongDivision() {
            const N_str = document.getElementById('dividend').value;
            const D_str = document.getElementById('divisor').value;

            const N = parseInt(N_str);
            const D = parseInt(D_str);

            const stepsOutput = document.getElementById('steps-output');
            const finalResult = document.getElementById('final-result');
            finalResult.classList.add('hidden');
            stepsOutput.innerHTML = '';

            // --- Input Validation ---
            if (isNaN(N) || N_str.length === 0 || N < 0) {
                return showMessage("Please enter a valid, non-negative Dividend.");
            }
            if (isNaN(D) || D_str.length === 0 || D <= 0) {
                return showMessage("Please enter a valid, positive Divisor.");
            }
            if (N_str.length > 9) {
                return showMessage("The dividend is too large (max 9 digits) for this simple demo.");
            }

            // --- Core Logic Setup ---
            let dividend = N;
            let divisor = D;
            let quotient = 0;
            let remainder = 0;

            const N_digits = N_str.split('').map(Number);
            let currentPartial = 0; // The number being divided in the current step
            let quotient_digits = [];
            let steps = [];
            let currentDivisorIndex = 0;

            // --- Visual Setup (Mimicking the division bar) ---
            steps.push(`<div class="text-center">`);
            steps.push(`<div class="division-setup"><span id="final-quotient" class="font-bold text-base text-green-700"></span>${N_str}</div>`);
            steps.push(`</div><br/>`);

            // --- Step-by-Step Algorithm ---
            for (let i = 0; i < N_digits.length; i++) {
                currentPartial = currentPartial * 10 + N_digits[i];

                // Check if current partial is large enough to divide
                if (currentPartial < divisor && quotient_digits.length > 0) {
                    // Add a zero to the quotient if we can't divide yet, but have already started
                    quotient_digits.push(0);
                    steps.push(`<div class="step-line text-sm text-gray-700">→ Current Partial: <span class="font-semibold">${currentPartial}</span> (Too small, bringing down next digit)</div>`);
                    continue;
                } else if (currentPartial < divisor && quotient_digits.length === 0) {
                    // If we're at the start and the first digit is too small, just continue.
                    steps.push(`<div class="step-line text-sm text-gray-700">→ Starting Partial: <span class="font-semibold">${currentPartial}</span> (Too small, bringing down next digit)</div>`);
                    continue;
                }

                // 1. Divide: Calculate the quotient digit (how many times D goes into currentPartial)
                let q_digit = Math.floor(currentPartial / divisor);
                
                // 2. Multiply: The product to be subtracted
                let product = q_digit * divisor;
                
                // 3. Subtract: Calculate the new remainder
                let rem = currentPartial - product;
                
                // Record the step
                steps.push(`<div class="step-line text-sm bg-blue-50 rounded-md p-2 mb-1">`);
                steps.push(`  <p><strong>Step ${quotient_digits.length + 1}:</strong></p>`);
                steps.push(`  <p class="ml-4">a. Divide: Current partial <span class="font-semibold">${currentPartial}</span> $\\div$ ${divisor} = <span class="font-semibold">${q_digit}</span></p>`);
                steps.push(`  <p class="ml-4">b. Multiply: ${q_digit} $\\times$ ${divisor} = <span class="font-semibold">${product}</span></p>`);
                steps.push(`  <p class="ml-4">c. Subtract: ${currentPartial} $-$ ${product} = <span class="font-semibold">${rem}</span></p>`);
                steps.push(`  <p class="ml-4">d. New Remainder: <span class="font-semibold">${rem}</span></p>`);

                if (i < N_digits.length - 1) {
                    steps.push(`  <p class="ml-4">e. Bring Down: Next digit is <span class="font-semibold">${N_digits[i+1]}</span>. New Partial: <span class="font-semibold">${rem * 10 + N_digits[i+1]}</span></p>`);
                } else {
                    steps.push(`  <p class="ml-4">e. End: No more digits to bring down.</p>`);
                }
                steps.push(`</div>`);


                // Update the state for the next iteration
                quotient_digits.push(q_digit);
                currentPartial = rem; // The remainder becomes the start of the next partial dividend
            }

            // --- Final Results Calculation ---
            quotient = parseInt(quotient_digits.join(''));
            remainder = currentPartial;

            // Update the steps output
            stepsOutput.innerHTML = steps.join('');

            // Update the quotient display above the dividend
            document.getElementById('final-quotient').textContent = quotient;

            // Update the final result display
            finalResult.innerHTML = `Result: <span class="text-green-600">${N} $\\div$ ${D}</span><br/>Quotient: <span class="text-green-600">${quotient}</span>, Remainder: <span class="text-green-600">${remainder}</span>`;
            finalResult.classList.remove('hidden');
        }

        // Initialize the app on load to show a default calculation
        window.onload = function() {
            // Wait a moment to ensure all elements are rendered before calculating
            setTimeout(calculateLongDivision, 100); 
        };
    </script>
</body>
</html>
