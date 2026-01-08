# Writing-Marathon-Leaderboard
Gamify your student's writing goals! Great for NaNaWriMo. Need to connect to Google Sheet with script - see ReadMe. 

Google Sheet Script Info:
Step 1: Prepare the Google Sheet
- Create a new Google Sheet and name it something like "Writing Marathon".
- In cell A1, type Name. In cell B1, type TotalWords.
- (Optional) You can pre-fill the names of your students in Column A and put 0 in Column B for each. If you don't, the script is smart enough to add them automatically the first time they submit words.

Step 2: Install the Apps Script
- Inside your Google Sheet, go to Extensions > Apps Script.
- Delete any code currently in the editor and paste the Google Apps Script at bottom of these instructions.
- Click the Save icon (floppy disk) and name the project "Marathon Backend".

Step 3: Deploy as a Web App (Critical Step)
- You need to make this script "public" so Canvas can talk to it.
- Click the blue Deploy button > New deployment.
- Select type as "Web app".
- Description: "Leaderboard API".
- Execute as: "Me" (your email).
- Who has access: Change this to "Anyone". Note: Don't worry, this doesn't give people access to your Google Drive; it only lets the code update that specific sheet.
- Click Deploy. Google will ask you to "Authorize access." Click through the prompts (you may need to click "Advanced" and then "Go to Marathon Backend (unsafe)" to proceed).
- Copy the Web App URL provided at the end. It should end in /exec.

Step 4: Connect the HTML to the Script
- Open your HTML code (After the Google Sheets Script below - look after the dotted lines below and pay attention to those section titles!).
- Look for this line (around line 258): const SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE';
- Replace the placeholder text with the URL you copied in Step 3. Keep the single quotes!

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
Google Apps Script - Copy and Paste into Script Page - DO NOT INCLUDE THE HTML AT THE BOTTOM OF THE PAGE, MAKE SURE TO STOP BEFORE THAT. Look for the dotted line!
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
// Writing Marathon Leaderboard - Google Apps Script
// SETUP INSTRUCTIONS:
// 1. Create a Google Sheet with columns: Name | TotalWords
// 2. Add student names in column A (starting at A2)
// 3. Set initial word counts to 0 in column B
// 4. Tools > Script editor > paste this code
// 5. Deploy > New deployment > Web app
// 6. Execute as: Me
// 7. Who has access: Anyone
// 8. Copy the web app URL

function doGet(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = sheet.getDataRange().getValues();
    
    // Skip header row, build leaderboard array
    const leaderboard = [];
    for (let i = 1; i < data.length; i++) {
      if (data[i][0]) { // Only if name exists
        leaderboard.push({
          name: data[i][0],
          totalWords: data[i][1] || 0
        });
      }
    }
    
    // Sort by totalWords descending
    leaderboard.sort((a, b) => b.totalWords - a.totalWords);
    
    return ContentService
      .createTextOutput(JSON.stringify({ success: true, leaderboard: leaderboard }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doPost(e) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = sheet.getDataRange().getValues();
    
    // Parse incoming data
    const params = JSON.parse(e.postData.contents);
    const studentName = params.name;
    const wordsToAdd = parseInt(params.words) || 0;
    
    if (!studentName || wordsToAdd <= 0) {
      return ContentService
        .createTextOutput(JSON.stringify({ success: false, error: "Invalid input" }))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    // Find student row
    let studentRow = -1;
    for (let i = 1; i < data.length; i++) {
      if (data[i][0] === studentName) {
        studentRow = i + 1; // Sheet rows are 1-indexed
        break;
      }
    }
    
    if (studentRow === -1) {
      // Student not found - add new row
      sheet.appendRow([studentName, wordsToAdd]);
    } else {
      // Update existing total
      const currentTotal = data[studentRow - 1][1] || 0;
      const newTotal = currentTotal + wordsToAdd;
      sheet.getRange(studentRow, 2).setValue(newTotal);
    }
    
    return ContentService
      .createTextOutput(JSON.stringify({ success: true, message: "Words added successfully!" }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
---------------------------------------------------------------------------------------------------------------------------------------------
HTML Code for Leaderboard
---------------------------------------------------------------------------------------------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Writing Marathon Leaderboard</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    
    body {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      padding: 20px;
      min-height: 100vh;
    }
    
    .container {
      max-width: 900px;
      margin: 0 auto;
    }
    
    .header {
      text-align: center;
      color: white;
      margin-bottom: 30px;
    }
    
    .header h1 {
      font-size: 2.5rem;
      font-weight: 700;
      margin-bottom: 10px;
      text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
    }
    
    .header p {
      font-size: 1.1rem;
      opacity: 0.95;
    }
    
    .submission-card {
      background: white;
      border-radius: 12px;
      padding: 25px;
      margin-bottom: 25px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.2);
    }
    
    .submission-card h2 {
      color: #333;
      margin-bottom: 20px;
      font-size: 1.3rem;
    }
    
    .form-group {
      margin-bottom: 15px;
    }
    
    .form-group label {
      display: block;
      margin-bottom: 8px;
      color: #555;
      font-weight: 600;
      font-size: 0.95rem;
    }
    
    .form-group input,
    .form-group select {
      width: 100%;
      padding: 12px 15px;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      font-size: 1rem;
      font-family: 'Inter', sans-serif;
      transition: border-color 0.3s;
    }
    
    .form-group input:focus,
    .form-group select:focus {
      outline: none;
      border-color: #667eea;
    }
    
    .submit-btn {
      width: 100%;
      padding: 14px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 1.1rem;
      font-weight: 600;
      cursor: pointer;
      transition: transform 0.2s, box-shadow 0.2s;
    }
    
    .submit-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
    }
    
    .submit-btn:active {
      transform: translateY(0);
    }
    
    .submit-btn:disabled {
      opacity: 0.6;
      cursor: not-allowed;
      transform: none;
    }
    
    .leaderboard-card {
      background: white;
      border-radius: 12px;
      padding: 25px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.2);
    }
    
    .leaderboard-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
    }
    
    .leaderboard-header h2 {
      color: #333;
      font-size: 1.5rem;
    }
    
    .refresh-btn {
      padding: 8px 16px;
      background: #667eea;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 0.9rem;
      font-weight: 600;
      transition: background 0.3s;
    }
    
    .refresh-btn:hover {
      background: #5568d3;
    }
    
    .loading {
      text-align: center;
      padding: 40px;
      color: #999;
      font-size: 1rem;
    }
    
    .loading::after {
      content: '...';
      animation: dots 1.5s infinite;
    }
    
    @keyframes dots {
      0%, 20% { content: '.'; }
      40% { content: '..'; }
      60%, 100% { content: '...'; }
    }
    
    .leaderboard-entry {
      display: grid;
      grid-template-columns: 60px 1fr 120px;
      gap: 15px;
      align-items: center;
      padding: 20px;
      border-bottom: 1px solid #f0f0f0;
      transition: all 0.5s ease;
    }
    
    .leaderboard-entry:last-child {
      border-bottom: none;
    }
    
    .leaderboard-entry.rank-1 {
      background: linear-gradient(90deg, #ffd700 0%, #fff 100%);
    }
    
    .leaderboard-entry.rank-2 {
      background: linear-gradient(90deg, #c0c0c0 0%, #fff 100%);
    }
    
    .leaderboard-entry.rank-3 {
      background: linear-gradient(90deg, #cd7f32 0%, #fff 100%);
    }
    
    /* Champion Mode Styling */
    .leaderboard-entry.champion {
      border: 3px solid #ffd700;
      border-radius: 8px;
      box-shadow: 0 0 20px rgba(255, 215, 0, 0.4);
      animation: championGlow 2s ease-in-out infinite;
    }
    
    @keyframes championGlow {
      0%, 100% {
        box-shadow: 0 0 20px rgba(255, 215, 0, 0.4);
      }
      50% {
        box-shadow: 0 0 30px rgba(255, 215, 0, 0.6);
      }
    }
    
    .rank {
      font-size: 1.8rem;
      font-weight: 700;
      color: #667eea;
      text-align: center;
    }
    
    .rank-1 .rank { color: #b8860b; }
    .rank-2 .rank { color: #6b7280; }
    .rank-3 .rank { color: #92400e; }
    
    .student-info h3 {
      font-size: 1.1rem;
      color: #333;
      margin-bottom: 8px;
      display: flex;
      align-items: center;
      gap: 8px;
      flex-wrap: wrap;
    }
    
    /* Champion Badge */
    .champion-badge {
      display: inline-flex;
      align-items: center;
      gap: 4px;
      padding: 4px 10px;
      background: linear-gradient(135deg, #ffd700 0%, #ffed4e 100%);
      color: #8b6914;
      font-size: 0.75rem;
      font-weight: 700;
      border-radius: 12px;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      box-shadow: 0 2px 8px rgba(255, 215, 0, 0.3);
    }
    
    .progress-container {
      background: #f0f0f0;
      border-radius: 10px;
      height: 8px;
      overflow: hidden;
      position: relative;
    }
    
    .progress-bar {
      height: 100%;
      background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
      border-radius: 10px;
      transition: width 0.8s ease;
    }
    
    .progress-bar.champion-progress {
      background: linear-gradient(90deg, #ffd700 0%, #ffed4e 100%);
    }
    
    .progress-text {
      font-size: 0.85rem;
      color: #666;
      margin-top: 4px;
    }
    
    .progress-text.goal-achieved {
      color: #2d3748;
      font-weight: 700;
    }
    
    .word-count {
      text-align: right;
    }
    
    .word-count .total {
      font-size: 1.5rem;
      font-weight: 700;
      color: #667eea;
    }
    
    .word-count .label {
      font-size: 0.85rem;
      color: #999;
      text-transform: uppercase;
      letter-spacing: 0.5px;
    }
    
    .message {
      padding: 12px 20px;
      border-radius: 8px;
      margin-top: 15px;
      font-weight: 500;
      text-align: center;
    }
    
    .message.success {
      background: #d4edda;
      color: #155724;
      border: 1px solid #c3e6cb;
    }
    
    .message.error {
      background: #f8d7da;
      color: #721c24;
      border: 1px solid #f5c6cb;
    }
    
    @media (max-width: 600px) {
      .header h1 {
        font-size: 1.8rem;
      }
      
      .leaderboard-entry {
        grid-template-columns: 50px 1fr 100px;
        gap: 10px;
        padding: 15px;
      }
      
      .rank {
        font-size: 1.4rem;
      }
      
      .student-info h3 {
        font-size: 1rem;
      }
      
      .word-count .total {
        font-size: 1.2rem;
      }
      
      .champion-badge {
        font-size: 0.7rem;
        padding: 3px 8px;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>📝 Writing Marathon</h1>
      <p>Race to 50,000 words!</p>
    </div>
    
    <div class="submission-card">
      <h2>Submit Today's Words</h2>
      <form id="submissionForm">
        <div class="form-group">
          <label for="studentName">Your Name</label>
          <input type="text" id="studentName" placeholder="Enter your name" required>
        </div>
        
        <div class="form-group">
          <label for="wordsWritten">Words Written Today</label>
          <input type="number" id="wordsWritten" min="1" placeholder="0" required>
        </div>
        
        <button type="submit" class="submit-btn" id="submitBtn">
          Add My Words 🚀
        </button>
        
        <div id="message"></div>
      </form>
    </div>
    
    <div class="leaderboard-card">
      <div class="leaderboard-header">
        <h2>🏆 Leaderboard</h2>
        <button class="refresh-btn" onclick="loadLeaderboard()">↻ Refresh</button>
      </div>
      
      <div id="leaderboard">
        <div class="loading">Loading rankings</div>
      </div>
    </div>
  </div>

  <script>
    // Google Apps Script Web App URL
    const SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE';
    
    const GOAL = 50000;
    
    // Track current totals to detect champion achievements
    let currentLeaderboard = [];
    
    // Load leaderboard on page load
    document.addEventListener('DOMContentLoaded', () => {
      loadLeaderboard();
    });
    
    // Handle form submission
    document.getElementById('submissionForm').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const studentName = document.getElementById('studentName').value.trim();
      const wordsWritten = parseInt(document.getElementById('wordsWritten').value);
      const submitBtn = document.getElementById('submitBtn');
      const messageDiv = document.getElementById('message');
      
      if (!studentName || wordsWritten <= 0) {
        showMessage('Please enter valid information', 'error');
        return;
      }
      
      // Check if this submission will push them over the goal
      const currentStudent = currentLeaderboard.find(s => s.name === studentName);
      const currentTotal = currentStudent ? currentStudent.totalWords : 0;
      const newTotal = currentTotal + wordsWritten;
      const wasUnderGoal = currentTotal < GOAL;
      const isNowOverGoal = newTotal >= GOAL;
      const achievedChampion = wasUnderGoal && isNowOverGoal;
      
      // Disable button and show loading
      submitBtn.disabled = true;
      submitBtn.textContent = 'Submitting...';
      messageDiv.innerHTML = '';
      
      try {
        const response = await fetch(SCRIPT_URL, {
          method: 'POST',
          mode: 'no-cors',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            name: studentName,
            words: wordsWritten
          })
        });
        
        // Show success message
        if (achievedChampion) {
          showMessage(`🎉 CONGRATULATIONS! You've reached ${newTotal.toLocaleString()} words and become a CHAMPION!`, 'success');
          
          // Triple confetti burst for champion achievement
          tripleConfettiBurst();
        } else {
          showMessage(`🎉 ${wordsWritten} words added successfully!`, 'success');
          
          // Single confetti burst
          confetti({
            particleCount: 100,
            spread: 70,
            origin: { y: 0.6 }
          });
        }
        
        // Clear form
        document.getElementById('wordsWritten').value = '';
        
        // Reload leaderboard after 1 second
        setTimeout(() => {
          loadLeaderboard();
        }, 1000);
        
      } catch (error) {
        showMessage('Error submitting words. Please try again.', 'error');
      } finally {
        submitBtn.disabled = false;
        submitBtn.textContent = 'Add My Words 🚀';
      }
    });
    
    // Triple confetti burst for champion achievement
    function tripleConfettiBurst() {
      const burst = (delay) => {
        setTimeout(() => {
          confetti({
            particleCount: 150,
            spread: 90,
            origin: { y: 0.6 },
            colors: ['#ffd700', '#ffed4e', '#ffa500', '#ff6347']
          });
        }, delay);
      };
      
      burst(0);
      burst(300);
      burst(600);
    }
    
    async function loadLeaderboard() {
      const leaderboardDiv = document.getElementById('leaderboard');
      leaderboardDiv.innerHTML = '<div class="loading">Updating rankings</div>';
      
      try {
        const response = await fetch(SCRIPT_URL);
        const data = await response.json();
        
        if (data.success && data.leaderboard) {
          currentLeaderboard = data.leaderboard;
          renderLeaderboard(data.leaderboard);
        } else {
          leaderboardDiv.innerHTML = '<div class="loading">No data available</div>';
        }
      } catch (error) {
        leaderboardDiv.innerHTML = '<div class="loading">Error loading leaderboard</div>';
        console.error('Error:', error);
      }
    }
    
    function renderLeaderboard(leaderboard) {
      const leaderboardDiv = document.getElementById('leaderboard');
      
      if (leaderboard.length === 0) {
        leaderboardDiv.innerHTML = '<div class="loading">No entries yet. Be the first!</div>';
        return;
      }
      
      let html = '';
      leaderboard.forEach((student, index) => {
        const rank = index + 1;
        const progress = Math.min((student.totalWords / GOAL) * 100, 100);
        const rankClass = rank <= 3 ? `rank-${rank}` : '';
        const isChampion = student.totalWords >= GOAL;
        const championClass = isChampion ? 'champion' : '';
        const progressBarClass = isChampion ? 'champion-progress' : '';
        const progressTextClass = isChampion ? 'goal-achieved' : '';
        
        // Determine progress text
        let progressText = `${progress.toFixed(1)}% of goal`;
        if (isChampion) {
          progressText = 'Goal Achieved! 🎉';
        }
        
        // Champion badge
        const championBadge = isChampion ? '<span class="champion-badge">🏆 Champion</span>' : '';
        
        html += `
          <div class="leaderboard-entry ${rankClass} ${championClass}">
            <div class="rank">#${rank}</div>
            <div class="student-info">
              <h3>
                ${escapeHtml(student.name)}
                ${championBadge}
              </h3>
              <div class="progress-container">
                <div class="progress-bar ${progressBarClass}" style="width: ${progress}%"></div>
              </div>
              <div class="progress-text ${progressTextClass}">${progressText}</div>
            </div>
            <div class="word-count">
              <div class="total">${student.totalWords.toLocaleString()}</div>
              <div class="label">Words</div>
            </div>
          </div>
        `;
      });
      
      leaderboardDiv.innerHTML = html;
    }
    
    function showMessage(text, type) {
      const messageDiv = document.getElementById('message');
      messageDiv.className = `message ${type}`;
      messageDiv.textContent = text;
      
      setTimeout(() => {
        messageDiv.innerHTML = '';
      }, 5000);
    }
    
    function escapeHtml(text) {
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }
  </script>
</body>
</html>
