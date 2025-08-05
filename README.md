<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>스마트 풋살 팀 편성</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800 p-4 min-h-screen flex flex-col items-center">
    
    <div class="container mx-auto p-6 bg-white rounded-xl shadow-lg w-full max-w-2xl">
        <h1 class="text-3xl font-bold text-center mb-6">스마트 풋살 팀 편성</h1>

        <!-- 1. 참가 인원 파악 섹션 -->
        <div class="mb-8 p-6 bg-gray-50 rounded-lg">
            <h2 class="text-xl font-semibold mb-4">1. 참가 인원 파악</h2>
            <div class="flex items-center space-x-4 mb-4">
                <input type="file" id="imageUpload" accept="image/*" class="flex-1 text-sm text-gray-500
                    file:mr-4 file:py-2 file:px-4
                    file:rounded-full file:border-0
                    file:text-sm file:font-semibold
                    file:bg-blue-50 file:text-blue-700
                    hover:file:bg-blue-100 cursor-pointer">
                <button onclick="uploadImage()" class="px-6 py-2 bg-blue-600 text-white font-semibold rounded-full shadow hover:bg-blue-700 transition duration-300">인원 파악</button>
            </div>
            <!-- 수동으로 인원 추가 기능 -->
            <div class="flex items-center space-x-2">
                <input type="text" id="manualPlayerInput" placeholder="이름을 입력하세요" class="flex-1 p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500">
                <button onclick="addPlayerManually()" class="px-6 py-2 bg-purple-600 text-white font-semibold rounded-full shadow hover:bg-purple-700 transition duration-300">수동으로 추가</button>
            </div>
            <p id="statusMessage" class="mt-4 text-gray-600"></p>
            <div id="playerList" class="mt-4 p-4 bg-white border border-gray-200 rounded-lg min-h-[5rem] flex flex-wrap gap-2">
                <p class="text-gray-400">인원 파악 버튼을 누르거나 이름을 직접 추가해주세요.</p>
            </div>
        </div>

        <!-- 2. 팀 설정 및 그룹 지정 섹션 -->
        <div class="mb-8 p-6 bg-gray-50 rounded-lg">
            <h2 class="text-xl font-semibold mb-4">2. 팀 설정 및 그룹 지정</h2>
            <div class="mb-4">
                <label for="teamCount" class="block text-sm font-medium text-gray-700 mb-1">팀 개수:</label>
                <select id="teamCount" class="block w-full p-2 border border-gray-300 rounded-md shadow-sm">
                    <option value="2">2팀</option>
                    <option value="3">3팀</option>
                    <option value="4">4팀</option>
                </select>
            </div>
            <div class="mb-4">
                <h3 class="font-medium text-lg mb-2">지인 그룹 지정</h3>
                <div class="flex space-x-2">
                    <input type="text" id="groupName" placeholder="그룹명 입력 (예: 00지인)" class="flex-1 p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500">
                    <button onclick="addGroup()" class="px-6 py-2 bg-green-600 text-white font-semibold rounded-full shadow hover:bg-green-700 transition duration-300">그룹으로 묶기</button>
                </div>
            </div>
            <div id="groupList" class="mt-4 p-4 bg-white border border-gray-200 rounded-lg min-h-[4rem]">
                <p class="text-gray-400">그룹이 지정되지 않았습니다.</p>
            </div>
        </div>

        <!-- 3. 팀 편성 및 결과 섹션 -->
        <div class="mb-8 p-6 bg-gray-50 rounded-lg">
            <h2 class="text-xl font-semibold mb-4">3. 팀 편성</h2>
            <button onclick="generateTeams()" class="w-full py-3 bg-indigo-600 text-white font-bold text-lg rounded-full shadow hover:bg-indigo-700 transition duration-300">팀 자동 편성</button>
            <div id="teamResults" class="mt-4 p-4 bg-white border border-gray-200 rounded-lg min-h-[10rem]">
                <p class="text-gray-400 text-center">여기에 팀 편성 결과가 표시됩니다.</p>
            </div>
        </div>
    </div>

    <script>
        // Global variables for managing state
        let allPlayers = [];
        let selectedPlayers = new Set();
        let groups = {};

        /**
         * Converts a file to a Base64 string.
         * @param {File} file The file to convert.
         * @returns {Promise<string>} The Base64 string of the file.
         */
        const fileToBase64 = (file) => {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result.split(',')[1]);
                reader.onerror = error => reject(error);
            });
        };

        /**
         * Uploads an image and uses the Gemini API to identify names.
         */
        async function uploadImage() {
            const fileInput = document.getElementById('imageUpload');
            const file = fileInput.files[0];
            const status = document.getElementById('statusMessage');
            
            if (!file) {
                status.textContent = '이미지 파일을 선택해주세요.';
                return;
            }

            status.textContent = '이미지 분석 중... 잠시만 기다려 주세요.';
            const playerList = document.getElementById('playerList');
            playerList.innerHTML = '<p class="text-gray-400">이미지를 분석하고 있습니다...</p>';

            try {
                const base64ImageData = await fileToBase64(file);
                
                // Construct the prompt for the Gemini API
                const prompt = "Please identify and list the names of the people from the image. Respond with a comma-separated list of names only, for example: 'Anderson, 국주, 김남혁'. Do not include any other text or explanation.";
                
                const chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
                const payload = {
                    contents: [{
                        role: "user",
                        parts: [
                            { text: prompt },
                            {
                                inlineData: {
                                    mimeType: file.type,
                                    data: base64ImageData
                                }
                            }
                        ]
                    }],
                };

                const apiKey = "AIzaSyCWpt1uxNSdV2B_QKV5B1G7Vd6osT3xCIc";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

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
                    const names = text.split(',').map(name => name.trim()).filter(name => name);
                    
                    allPlayers = [...new Set(names)]; // Remove duplicates
                    
                    if (allPlayers.length > 0) {
                        status.textContent = `${allPlayers.length}명의 인원을 파악했습니다.`;
                        renderPlayers();
                    } else {
                        status.textContent = '이미지에서 이름을 찾을 수 없습니다.';
                        playerList.innerHTML = '<p class="text-gray-400">이름을 찾을 수 없습니다. 다른 이미지를 사용해 보세요.</p>';
                    }
                } else {
                    status.textContent = '이미지 분석에 실패했습니다. 응답 형식이 올바르지 않습니다.';
                    console.error('API response error:', result);
                }

            } catch (error) {
                status.textContent = `통신 오류: ${error.message}`;
                console.error('Fetch error:', error);
            }
        }

        /**
         * Renders the list of players on the screen.
         */
        function renderPlayers() {
            const playerList = document.getElementById('playerList');
            playerList.innerHTML = ''; // Clear existing list
            if (allPlayers.length === 0) {
                playerList.innerHTML = '<p class="text-gray-400">인원 파악 버튼을 누르거나 이름을 직접 추가해주세요.</p>';
                return;
            }

            allPlayers.forEach(player => {
                const span = document.createElement('span');
                span.textContent = player;
                span.classList.add('px-4', 'py-1', 'bg-gray-200', 'rounded-full', 'cursor-pointer', 'hover:bg-blue-300', 'transition', 'duration-150');
                span.onclick = () => togglePlayerSelection(span, player);
                playerList.appendChild(span);
            });
        }

        /**
         * Toggles the selection state of a player.
         * @param {HTMLElement} element The player element to toggle.
         * @param {string} player The name of the player.
         */
        function togglePlayerSelection(element, player) {
            if (selectedPlayers.has(player)) {
                selectedPlayers.delete(player);
                element.classList.remove('bg-blue-600', 'text-white', 'scale-105');
                element.classList.add('bg-gray-200');
            } else {
                selectedPlayers.add(player);
                element.classList.remove('bg-gray-200');
                element.classList.add('bg-blue-600', 'text-white', 'scale-105');
            }
        }

        /**
         * Assigns selected players to a group.
         */
        function addGroup() {
            if (selectedPlayers.size === 0) {
                alert('그룹으로 묶을 인원을 선택해주세요.');
                return;
            }
            const groupNameInput = document.getElementById('groupName');
            const groupName = groupNameInput.value.trim();
            if (!groupName) {
                alert('그룹명을 입력해주세요.');
                return;
            }

            // Convert Set to array and store in groups
            groups[groupName] = Array.from(selectedPlayers);
            renderGroups();
            
            // Reset selection and input field after grouping
            selectedPlayers.clear();
            document.querySelectorAll('#playerList span').forEach(el => {
                el.classList.remove('bg-blue-600', 'text-white', 'scale-105');
                el.classList.add('bg-gray-200');
            });
            groupNameInput.value = '';
        }
        
        /**
         * Manually adds a player to the list.
         */
        function addPlayerManually() {
            const manualInput = document.getElementById('manualPlayerInput');
            const playerName = manualInput.value.trim();

            if (playerName) {
                // Add the player if they don't already exist
                if (!allPlayers.includes(playerName)) {
                    allPlayers.push(playerName);
                    renderPlayers();
                    // Clear the input field after adding
                    manualInput.value = '';
                    document.getElementById('statusMessage').textContent = `"${playerName}"님이 명단에 추가되었습니다.`;
                } else {
                    document.getElementById('statusMessage').textContent = `"${playerName}"님은 이미 명단에 있습니다.`;
                }
            } else {
                document.getElementById('statusMessage').textContent = '추가할 이름을 입력해주세요.';
            }
        }

        /**
         * Renders the list of groups on the screen.
         */
        function renderGroups() {
            const groupList = document.getElementById('groupList');
            groupList.innerHTML = ''; // Clear existing list
            const groupNames = Object.keys(groups);
            if (groupNames.length === 0) {
                groupList.innerHTML = '<p class="text-gray-400">그룹이 지정되지 않았습니다.</p>';
                return;
            }

            groupNames.forEach(name => {
                const div = document.createElement('div');
                div.classList.add('p-2', 'bg-white', 'rounded-md', 'shadow-sm', 'border', 'border-gray-100');
                div.innerHTML = `<strong class="text-blue-700">${name}</strong>: ${groups[name].join(', ')}`;
                groupList.appendChild(div);
            });
        }

        /**
         * Generates teams based on the configured settings, ensuring equal team sizes.
         */
        function generateTeams() {
            if (allPlayers.length === 0) {
                alert('먼저 참가 인원을 파악해주세요.');
                return;
            }

            const teamCount = parseInt(document.getElementById('teamCount').value, 10);
            const teamResults = document.getElementById('teamResults');
            teamResults.innerHTML = '<p class="text-gray-600 text-center">팀 편성 중...</p>';

            // 팀 편성 로직
            const teams = Array.from({ length: teamCount }, () => []);
            let remainingPlayers = [...allPlayers];

            // 1. 그룹에 속한 인원들을 균등하게 배정 (그룹이 큰 순서대로)
            const groupsArray = Object.values(groups).sort((a, b) => b.length - a.length);

            for (const group of groupsArray) {
                // 현재 인원수가 가장 적은 팀을 찾아 그룹 배정
                const minTeamIndex = teams.reduce((minIndex, currentTeam, currentIndex, arr) => {
                    return currentTeam.length < arr[minIndex].length ? currentIndex : minIndex;
                }, 0);
                
                teams[minTeamIndex].push(...group);
                
                // 배정된 멤버를 남은 인원에서 제거
                remainingPlayers = remainingPlayers.filter(player => !group.includes(player));
            }

            // 2. 남은 인원들을 무작위로 섞어 균등하게 분배
            remainingPlayers.sort(() => Math.random() - 0.5);
            remainingPlayers.forEach((player, index) => {
                // 다시 한 번 현재 인원수가 가장 적은 팀에 배정
                const minTeamIndex = teams.reduce((minIndex, currentTeam, currentIndex, arr) => {
                    return currentTeam.length < arr[minIndex].length ? currentIndex : minIndex;
                }, 0);
                teams[minTeamIndex].push(player);
            });

            renderTeams(teams);
        }

        /**
         * Renders the generated teams on the screen.
         * @param {Array<Array<string>>} teams An array of arrays, where each inner array represents a team.
         */
        function renderTeams(teams) {
            const teamResults = document.getElementById('teamResults');
            teamResults.innerHTML = '';

            if (teams.length === 0) {
                teamResults.innerHTML = '<p class="text-gray-400 text-center">팀 편성 결과가 없습니다.</p>';
                return;
            }

            teams.forEach((team, index) => {
                const teamDiv = document.createElement('div');
                teamDiv.classList.add('team-box', 'bg-white', 'rounded-lg', 'shadow', 'p-4', 'mb-2');
                teamDiv.innerHTML = `
                    <h3 class="font-bold text-xl text-indigo-700">팀 ${index + 1} <span class="text-sm font-normal text-gray-500">(${team.length}명)</span></h3>
                    <p class="mt-2 text-gray-700">${team.join(', ')}</p>
                `;
                teamResults.appendChild(teamDiv);
            });
        }
    </script>
</body>
</html>
