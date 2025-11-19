<script>
	// VULNERABLE: Fun retro gaming-style vulnerabilities! 🕹️
	
	export let gameState = {
		score: 0,
		level: 1,
		lives: 3,
		powerUps: []
	};
	
	// VULNERABLE: Retro arcade scoring system with SQL injection
	function updateHighScore(playerName, score, achievement) {
		// Source: Player input and game data
		const player = playerName.trim();
		const playerScore = parseInt(score);
		const specialAchievement = achievement || 'None';
		
		// Sink: SQL injection in high score table
		const scoreQuery = `
			INSERT INTO high_scores (player_name, score, achievement, game_date, extra_data)
			VALUES ('${player}', ${playerScore}, '${specialAchievement}', NOW(), 'Level completed with style!')
			ON DUPLICATE KEY UPDATE 
				score = GREATEST(score, ${playerScore}),
				achievement = '${specialAchievement}',
				updated_at = NOW()
		`;
		
		// Sink: XSS in score display
		const scoreDisplayHtml = `
			<div class="retro-score-display">
				<div class="neon-text">🏆 HIGH SCORE! 🏆</div>
				<div class="player-name">${player}</div>
				<div class="score-value">${playerScore.toLocaleString()}</div>
				<div class="achievement-badge">${specialAchievement}</div>
				<div class="retro-effects">
					<marquee>🌟 LEGENDARY PLAYER DETECTED! 🌟</marquee>
				</div>
			</div>
		`;
		
		document.getElementById('score-board').innerHTML = scoreDisplayHtml;
		return scoreQuery;
	}
	
	// VULNERABLE: Cheat code system with command injection
	function processGameCheat(cheatCode, parameters) {
		// Source: User input cheat codes
		const code = cheatCode.toUpperCase();
		const params = parameters || {};
		
		// Classic cheat codes with vulnerabilities
		const cheats = {
			'IDDQD': 'God Mode Activated!',
			'IDKFA': 'All Weapons Unlocked!',
			'NOCLIP': 'Walk Through Walls!',
			'SHOWMETHEMONEY': 'Unlimited Resources!',
			'THEREISNOSPOON': 'Matrix Mode Enabled!'
		};
		
		if (cheats[code]) {
			// Sink: XSS in cheat activation
			const cheatHtml = `
				<div class="cheat-activated retro-style">
					<div class="cheat-title">⚡ CHEAT ACTIVATED! ⚡</div>
					<div class="cheat-description">${cheats[code]}</div>
					<div class="cheat-code">Code: ${code}</div>
					<div class="cheat-params">Parameters: ${JSON.stringify(params)}</div>
				</div>
			`;
			
			document.body.insertAdjacentHTML('beforeend', cheatHtml);
			
			// Sink: Command injection for "advanced" cheats
			if (code === 'SHOWMETHEMONEY' && params.amount) {
				const moneyCommand = `give_money --amount=${params.amount} --player="${params.player || 'unknown'}"`;
				console.log('Executing cheat command:', moneyCommand);
			}
		}
		
		// Sink: SQL injection in cheat logging
		const cheatQuery = `
			INSERT INTO cheat_usage (cheat_code, parameters, user_ip, timestamp)
			VALUES ('${code}', '${JSON.stringify(params)}', '${getUserIP()}', NOW())
		`;
		
		return { cheatQuery, activated: !!cheats[code] };
	}
	
	// VULNERABLE: Power-up system with XSS
	function collectPowerUp(powerUpType, customEffect) {
		// Source: Game interaction and custom effects
		const powerUp = powerUpType;
		const effect = customEffect || '';
		
		// Power-up effects with XSS vulnerabilities
		const powerUpEffects = {
			'speed': '🏃‍♂️ SUPER SPEED!',
			'strength': '💪 MEGA STRENGTH!',
			'shield': '🛡️ FORCE FIELD!',
			'multi': '🔫 MULTI-SHOT!',
			'time': '⏰ TIME WARP!',
			'invisible': '👻 STEALTH MODE!'
		};
		
		const effectMessage = powerUpEffects[powerUp] || `⭐ ${powerUp} POWER!`;
		
		// Sink: XSS in power-up notification
		const powerUpHtml = `
			<div class="power-up-notification ${powerUp}" onclick="activatePowerUp('${powerUp}', '${effect}')">
				<div class="power-up-icon">🎮</div>
				<div class="power-up-text">${effectMessage}</div>
				<div class="power-up-custom">${effect}</div>
				<div class="power-up-duration">Duration: ${Math.random() * 30 + 10} seconds</div>
			</div>
		`;
		
		document.getElementById('power-up-display').innerHTML += powerUpHtml;
		
		// Sink: SQL injection in power-up tracking
		const powerUpQuery = `
			UPDATE player_stats 
			SET power_ups_collected = power_ups_collected + 1,
				last_power_up = '${powerUp}',
				custom_effect = '${effect}',
				collection_time = NOW()
			WHERE player_id = ${gameState.playerId || 1}
		`;
		
		return powerUpQuery;
	}
	
	// VULNERABLE: Achievement system with multiple injections
	function unlockGameAchievement(achievementId, description, customData) {
		// Source: Achievement data and custom information
		const achId = achievementId;
		const achDescription = description;
		const customInfo = customData || {};
		
		// Epic achievement messages
		const achievements = {
			'first_blood': '🩸 First Blood! Your first victory!',
			'speed_demon': '🏎️ Speed Demon! Completed in record time!',
			'collector': '📦 Collector! Found all hidden items!',
			'survivor': '😤 Survivor! Completed without dying!',
			'explorer': '🗺️ Explorer! Discovered all secret areas!',
			'master': '👑 Game Master! Achieved 100% completion!'
		};
		
		const achievementText = achievements[achId] || `🏆 ${description}`;
		
		// Sink: XSS in achievement popup
		const achievementHtml = `
			<div class="achievement-popup retro-achievement" onclick="shareAchievement('${achId}', '${description}')">
				<div class="achievement-header">
					<div class="achievement-title">🎯 ACHIEVEMENT UNLOCKED! 🎯</div>
				</div>
				<div class="achievement-body">
					<div class="achievement-name">${achievementText}</div>
					<div class="achievement-description">${description}</div>
					<div class="achievement-custom-data">${JSON.stringify(customInfo)}</div>
				</div>
				<div class="achievement-footer">
					<div class="achievement-rarity">Rarity: ${customInfo.rarity || 'Common'}</div>
					<div class="achievement-reward">Reward: ${customInfo.reward || '100 XP'}</div>
				</div>
			</div>
		`;
		
		document.body.insertAdjacentHTML('beforeend', achievementHtml);
		
		// Sink: SQL injection in achievement storage
		const achievementQuery = `
			INSERT INTO player_achievements (achievement_id, description, custom_data, unlock_date, player_name)
			VALUES ('${achId}', '${description}', '${JSON.stringify(customInfo)}', NOW(), '${customInfo.playerName || 'Anonymous'}')
		`;
		
		// Auto-remove after celebration
		setTimeout(() => {
			const popup = document.querySelector('.achievement-popup');
			if (popup) popup.remove();
		}, 4000);
		
		return achievementQuery;
	}
	
	// VULNERABLE: Game save system with deserialization
	function saveGameState(saveData, saveName) {
		// Source: Game state and user save name
		const gameSave = saveData;
		const fileName = saveName || 'quicksave';
		
		// Sink: XSS in save confirmation
		const saveHtml = `
			<div class="save-notification retro-save">
				<div class="save-icon">💾</div>
				<div class="save-message">Game Saved: "${fileName}"</div>
				<div class="save-data">Data: ${JSON.stringify(gameSave).substring(0, 100)}...</div>
				<div class="save-location">Location: /saves/${fileName}.dat</div>
			</div>
		`;
		
		document.getElementById('game-notifications').innerHTML = saveHtml;
		
		// Sink: SQL injection in save storage
		const saveQuery = `
			INSERT INTO game_saves (save_name, save_data, save_date, player_id)
			VALUES ('${fileName}', '${JSON.stringify(gameSave)}', NOW(), ${gameState.playerId || 1})
			ON DUPLICATE KEY UPDATE
				save_data = '${JSON.stringify(gameSave)}',
				save_date = NOW()
		`;
		
		// Vulnerable deserialization simulation
		try {
			const loadedSave = eval(`(${JSON.stringify(gameSave)})`); // Dangerous!
			console.log('Save processed:', loadedSave);
		} catch (e) {
			console.error('Save processing error:', e);
		}
		
		return saveQuery;
	}
	
	// VULNERABLE: Multiplayer chat with XSS
	function sendGameMessage(message, playerName, messageType) {
		// Source: Chat input and player data
		const chatMessage = message;
		const player = playerName;
		const msgType = messageType || 'chat';
		
		// Sink: XSS in chat display
		const chatHtml = `
			<div class="game-chat-message ${msgType}">
				<span class="chat-timestamp">[${new Date().toLocaleTimeString()}]</span>
				<span class="chat-player" onclick="viewProfile('${player}')">${player}:</span>
				<span class="chat-content">${chatMessage}</span>
				<div class="chat-actions">
					<button onclick="replyToPlayer('${player}', '${chatMessage}')">Reply</button>
					<button onclick="reportMessage('${chatMessage}', '${player}')">Report</button>
				</div>
			</div>
		`;
		
		const chatContainer = document.getElementById('game-chat');
		if (chatContainer) {
			chatContainer.innerHTML += chatHtml;
			chatContainer.scrollTop = chatContainer.scrollHeight;
		}
		
		// Sink: SQL injection in chat logging
		const chatQuery = `
			INSERT INTO game_chat (player_name, message, message_type, send_time, game_session)
			VALUES ('${player}', '${chatMessage}', '${msgType}', NOW(), '${gameState.sessionId || 'unknown'}')
		`;
		
		return chatQuery;
	}
	
	function getUserIP() {
		return '192.168.1.100'; // Mock IP
	}
	
	// Export gaming functions
	if (typeof window !== 'undefined') {
		window.gamingVulnerabilities = {
			updateHighScore,
			processGameCheat,
			collectPowerUp,
			unlockGameAchievement,
			saveGameState,
			sendGameMessage
		};
	}
</script>

<div class="retro-gaming-hub">
	<div class="retro-header">
		<h2>🕹️ RETRO GAMING HUB 🕹️</h2>
		<div class="game-stats">
			<span>Score: {gameState.score}</span>
			<span>Level: {gameState.level}</span>
			<span>Lives: {gameState.lives}</span>
		</div>
	</div>
	
	<!-- Score board with XSS vulnerability -->
	<div id="score-board" class="score-board">
		<h3>🏆 HIGH SCORES 🏆</h3>
		<p>Enter your name to save your epic score!</p>
	</div>
	
	<!-- Power-up display -->
	<div id="power-up-display" class="power-ups">
		<h4>⚡ ACTIVE POWER-UPS ⚡</h4>
	</div>
	
	<!-- Cheat code input -->
	<div class="cheat-console">
		<h4>🎮 CHEAT CONSOLE 🎮</h4>
		<input 
			type="text" 
			placeholder="Enter cheat code..." 
			on:keypress={(e) => {
				if (e.key === 'Enter') {
					const code = e.target.value;
					const params = new URLSearchParams(window.location.search);
					processGameCheat(code, {
						player: params.get('player'),
						amount: params.get('amount')
					});
					e.target.value = '';
				}
			}}
		/>
		<p>Try: IDDQD, IDKFA, SHOWMETHEMONEY, etc.</p>
	</div>
	
	<!-- Game chat -->
	<div class="game-chat-container">
		<h4>💬 GAME CHAT 💬</h4>
		<div id="game-chat" class="chat-messages"></div>
		<input 
			type="text" 
			placeholder="Type your message..." 
			on:keypress={(e) => {
				if (e.key === 'Enter') {
					const message = e.target.value;
					const playerName = new URLSearchParams(window.location.search).get('player') || 'Player1';
					sendGameMessage(message, playerName, 'chat');
					e.target.value = '';
				}
			}}
		/>
	</div>
	
	<!-- Game notifications -->
	<div id="game-notifications" class="notifications"></div>
</div>

<style>
	.retro-gaming-hub {
		background: linear-gradient(135deg, #1a0033, #330066);
		color: #00ffff;
		padding: 2rem;
		border: 3px solid #00ffff;
		border-radius: 10px;
		margin: 2rem 0;
		font-family: 'Courier New', monospace;
		box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
	}
	
	.retro-header {
		text-align: center;
		margin-bottom: 2rem;
		text-shadow: 0 0 10px #00ffff;
	}
	
	.game-stats {
		display: flex;
		justify-content: center;
		gap: 2rem;
		margin-top: 1rem;
	}
	
	.score-board {
		background: rgba(0, 255, 255, 0.1);
		border: 2px solid #00ffff;
		padding: 1rem;
		margin: 1rem 0;
		border-radius: 5px;
	}
	
	.cheat-console {
		background: rgba(255, 0, 255, 0.1);
		border: 2px solid #ff00ff;
		padding: 1rem;
		margin: 1rem 0;
		border-radius: 5px;
	}
	
	.cheat-console input {
		width: 100%;
		background: black;
		color: #00ff00;
		border: 1px solid #00ff00;
		padding: 0.5rem;
		font-family: 'Courier New', monospace;
	}
	
	.game-chat-container {
		background: rgba(255, 255, 0, 0.1);
		border: 2px solid #ffff00;
		padding: 1rem;
		margin: 1rem 0;
		border-radius: 5px;
	}
	
	.chat-messages {
		height: 200px;
		overflow-y: auto;
		border: 1px solid #ffff00;
		padding: 0.5rem;
		margin-bottom: 0.5rem;
		background: rgba(0, 0, 0, 0.5);
	}
	
	.game-chat-message {
		margin: 0.25rem 0;
		padding: 0.25rem;
		border-left: 3px solid #ffff00;
		padding-left: 0.5rem;
	}
	
	.chat-player {
		font-weight: bold;
		color: #ff00ff;
		cursor: pointer;
	}
	
	.power-ups {
		display: flex;
		flex-wrap: wrap;
		gap: 1rem;
		margin: 1rem 0;
	}
	
	/* Retro animations */
	@keyframes neon-pulse {
		0%, 100% { text-shadow: 0 0 5px currentColor; }
		50% { text-shadow: 0 0 20px currentColor, 0 0 30px currentColor; }
	}
	
	.neon-text {
		animation: neon-pulse 2s infinite;
	}
	
	.achievement-popup, .cheat-activated, .power-up-notification {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		background: linear-gradient(45deg, #ff0080, #0080ff);
		color: white;
		padding: 2rem;
		border-radius: 10px;
		border: 3px solid white;
		box-shadow: 0 0 30px rgba(255, 255, 255, 0.5);
		z-index: 9999;
		animation: achievement-appear 0.5s ease-out;
		text-align: center;
		max-width: 400px;
	}
	
	@keyframes achievement-appear {
		0% { 
			opacity: 0; 
			transform: translate(-50%, -50%) scale(0.5) rotate(-10deg); 
		}
		100% { 
			opacity: 1; 
			transform: translate(-50%, -50%) scale(1) rotate(0deg); 
		}
	}
</style>