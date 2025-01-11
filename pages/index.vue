<template>
    <div class="game-container">
      <!-- Message de bienvenue -->
      <div class="welcome-message">
        <h1>Bienvenue au Jeu de Mémoire !</h1>
        <p>Testez votre mémoire et amusez-vous à retrouver toutes les paires de cartes.</p>
      </div>
  
      <!-- Section des règles du jeu -->
      <div class="rules-section">
        <h2>Règles du jeu :</h2>
        <ul>
          <li>Sélectionnez un niveau pour commencer.</li>
          <li>Retournez deux cartes à la fois pour trouver des paires correspondantes.</li>
          <li>Terminez avant la fin du chronomètre pour gagner !</li>
          <li>Après avoir terminé un niveau, passez automatiquement au niveau suivant.</li>
        </ul>
      </div>
  
      <!-- Choix du niveau -->
      <div class="level-selector" v-if="!gameStarted">
        <label for="level">Choisissez un niveau :</label>
        <select id="level" v-model="selectedLevel" class="styled-select">
          <option value="easy">Débutant</option>
          <option value="medium">Intermédiaire</option>
          <option value="hard">Expert</option>
        </select>
        <button @click="startGame">Commencer</button>
      </div>
  
      <!-- Chronomètre -->
      <div class="timer" v-if="gameStarted && !gameOver">Temps restant : {{ timer }}s</div>
  
      <!-- Plateau de jeu -->
      <div class="cards-grid" v-if="gameStarted">
        <div 
          v-for="(card, index) in shuffledCards" 
          :key="index" 
          class="card" 
          :class="{ flipped: selectedCards.includes(index) || matchedCards.includes(card.id) }"
          @click="flipCard(index)">
          <div class="card-inner">
            <div class="card-front"></div>
            <div class="card-back">{{ card.value }}</div>
          </div>
        </div>
      </div>
  
      <!-- Bouton pour annuler le jeu -->
      <button class="cancel-button" v-if="gameStarted" @click="cancelGame">Annuler le jeu</button>
  
      <!-- Messages de fin de jeu -->
      <p class="message" v-if="gameOver">🎉 Bravo ! Vous avez gagné et passez au niveau suivant !</p>
      <p class="message" v-if="timeUp">⏰ Temps écoulé ! Reessayez.</p>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        levels: {
          easy: 6,
          medium: 10,
          hard: 15,
        },
        cards: [],
        selectedCards: [],
        matchedCards: [],
        gameOver: false,
        gameStarted: false,
        selectedLevel: 'easy',
        timer: 60,
        timeUp: false,
        countdown: null,
      };
    },
    computed: {
      shuffledCards() {
        return [...this.cards, ...this.cards]
          .sort(() => Math.random() - 0.5)
          .map((card, index) => ({ ...card, index }));
      },
    },
    methods: {
      initializeCards() {
        const levelCount = this.levels[this.selectedLevel];
        const values = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('').slice(0, levelCount);
        this.cards = values.map((value, index) => ({ id: index + 1, value }));
      },
      startGame() {
        this.resetGameState(); // Réinitialiser l'état du jeu
        this.initializeCards();
        this.startTimer();
        this.gameStarted = true;
      },
      cancelGame() {
      this.resetGame(); // Appeler la méthode pour réinitialiser le jeu
    },
      resetGameState() {
        this.selectedCards = [];
        this.matchedCards = [];
        this.timeUp = false;
        this.gameOver = false;
        this.timer = this.selectedLevel === 'easy' ? 60 : this.selectedLevel === 'medium' ? 70 : 90; // Réinitialiser le timer selon le niveau
        clearInterval(this.countdown); // Arrêter le précédent timer
      },
      flipCard(index) {
        if (this.selectedCards.length < 2 && !this.selectedCards.includes(index)) {
          this.selectedCards.push(index);
          if (this.selectedCards.length === 2) {
            this.checkMatch();
          }
        }
      },
      checkMatch() {
        const [firstIndex, secondIndex] = this.selectedCards;
        const firstCard = this.shuffledCards[firstIndex];
        const secondCard = this.shuffledCards[secondIndex];
        if (firstCard.id === secondCard.id) {
          this.matchedCards.push(firstCard.id);
        }
        setTimeout(() => {
          this.selectedCards = [];
          if (this.matchedCards.length === this.cards.length) {
            this.endGame(true);
          }
        }, 1000);
      },
      startTimer() {
        this.countdown = setInterval(() => {
          if (this.timer > 0) {
            this.timer--;
          } else {
            this.endGame(false);
          }
        }, 1000);
      },
      endGame(won) {
        clearInterval(this.countdown);
        this.gameOver = won;
        this.timeUp = !won;
        if (won) {
          if (this.selectedLevel === 'easy') {
            this.selectedLevel = 'medium';
          } else if (this.selectedLevel === 'medium') {
            this.selectedLevel = 'hard';
          } else {
            this.resetGame(); // Réinitialiser le jeu après le niveau hard
            return;
          }
          this.startGame(); // Démarrer le nouveau niveau
        } else {
          this.gameStarted = false; // Fin du jeu si le temps est écoulé
        }
      },
      resetGame() {
        this.gameStarted = false;
        this.selectedLevel = 'easy'; // Réinitialiser le niveau
        this.cards = []; // Réinitialiser les cartes
        this.resetGameState(); // Réinitialiser l'état du jeu
      },
    },
  };
  </script>
  
  <style>
body {
  margin: 0;
  font-family: 'Arial', sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #3a357d, #D81B60);
  color: #fff;
  height: 100vh; 
}

template {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.game-container {
  max-width: 90vw; 
  width: 100%;
  max-height: 90vh;
  margin: 20px auto; 
  padding: 20px;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.2);
  color: #333;
  overflow-y: auto;
}

.welcome-message {
  margin-bottom: 20px;
}

.rules-section {
  background: #f9f9f9;
  padding: 15px;
  margin-bottom: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.rules-section h2 {
  margin-bottom: 10px;
}

.level-selector {
  margin-bottom: 20px;
}

.styled-select {
  padding: 10px;
  border: 2px solid #3a357d;
  border-radius: 8px;
  background: #f3f3f3;
  transition: border 0.3s;
}

.styled-select:focus {
  border-color: #D81B60;
  outline: none;
}

.timer {
  margin-bottom: 10px;
  font-size: 1.2rem;
  color: #333;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(80px, 1fr)); 
  gap: 8px;
}

.card {
  width: 100%;
  padding-top: 100%; 
  position: relative;
  perspective: 1000px;
  cursor: pointer;
}

.card-inner {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  transform-style: preserve-3d;
  transition: transform 0.5s ease-in-out;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front, .card-back {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  backface-visibility: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.5rem; 
  font-weight: bold;
  border-radius: 8px;
}

.card-front {
  background-color: #3a357d;
}

.card-back {
  background-color: #D81B60;
  color: white;
  transform: rotateY(180deg);
}

.message {
  margin-top: 20px;
  font-size: 1.5rem;
  color: #4caf50;
  animation: fadeIn 1s;
}

.cancel-button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #ff4d4d; 
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s;
}

.cancel-button:hover {
  background-color: #e63939; 
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Media Queries pour améliorer la responsivité */
@media (max-width: 600px) {
  .welcome-message h1 {
    font-size: 1.5rem;
  }

  .timer {
    font-size: 1rem;
  }

  .message {
    font-size: 1.2rem;
  }

  .styled-select {
    padding: 8px;
  }

  .card-front, .card-back {
    font-size: 1.2rem; 
  }
}
  </style>