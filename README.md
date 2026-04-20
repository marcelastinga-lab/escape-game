<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Escape Game: Sauvetage à Vertefeuille</title>
    <style>
        :root {
            --primary: #22c55e;
            --dark-green: #166534;
            --bg-light: #f0fdf4;
            --white: #ffffff;
            --error: #ef4444;
        }

        body { 
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; 
            background-color: var(--bg-light); 
            color: var(--dark-green); 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            min-height: 100vh; 
            margin: 0;
            overflow-x: hidden;
        }

        #game-box { 
            background: var(--white); 
            padding: 40px; 
            border-radius: 24px; 
            box-shadow: 0 20px 50px rgba(22, 101, 52, 0.15); 
            width: 90%; 
            max-width: 600px; 
            text-align: center; 
            border: 4px solid #bdf0c4;
            position: relative;
            transition: transform 0.3s ease;
        }

        .hidden { display: none; }

        /* Animation de secousse pour les erreurs */
        .shake { animation: shake 0.4s ease-in-out; }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            75% { transform: translateX(10px); }
        }

        /* Barre de progression */
        .progress-bar-container {
            width: 100%;
            height: 10px;
            background: #e2e8f0;
            border-radius: 5px;
            margin-bottom: 20px;
            overflow: hidden;
        }
        #progress-fill {
            height: 100%;
            background: var(--primary);
            width: 0%;
            transition: width 0.5s ease;
        }

        .robot-msg { 
            font-style: italic; 
            background: #dcfce7; 
            padding: 20px; 
            border-radius: 16px; 
            margin-bottom: 25px; 
            border-left: 8px solid var(--primary); 
            text-align: left;
            font-size: 1.05em;
        }

        h1 { color: var(--dark-green); margin-top: 0; }

        button { 
            background-color: var(--primary); 
            color: white; 
            border: none; 
            padding: 14px 28px; 
            border-radius: 12px; 
            cursor: pointer; 
            font-size: 16px; 
            margin: 10px auto; 
            display: block;
            width: 100%;
            transition: all 0.2s; 
            font-weight: bold;
            box-shadow: 0 4px 0 #15803d;
        }

        button:hover { 
            background-color: #16a34a; 
            transform: translateY(-2px);
            box-shadow: 0 6px 0 #15803d;
        }

        button:active {
            transform: translateY(2px);
            box-shadow: 0 0 0 #15803d;
        }

        #feedback { 
            margin-top: 20px; 
            font-weight: bold; 
            min-height: 24px;
            padding: 10px;
            border-radius: 8px;
        }
        .correct-anim { background: #dcfce7; color: #15803d; }
        .wrong-anim { background: #fee2e2; color: #dc2626; }

        .emoji-large { font-size: 3rem; display: block; margin-bottom: 10px; }
    </style>
</head>
<body>

<div id="game-box">
    <div id="screen-start">
        <span class="emoji-large">🌲</span>
        <h1>Mission Vertefeuille</h1>
        <p>Alerte ! Le système de sécurité de la forêt est verrouillé. Lumi le robot a besoin de tes connaissances en français pour redémarrer les serveurs.</p>
        <div class="robot-msg">🤖 <strong>Lumi:</strong> "Vite ! Chaque énigme résolue libère un verrou de sécurité !"</div>
        <button onclick="startGame()">DÉVERROUILLER LE SYSTÈME</button>
    </div>

    <div id="question-container" class="hidden">
        <div class="progress-bar-container">
            <div id="progress-fill"></div>
        </div>
        <div class="progress" id="progress-text">Mission 1 sur 10</div>
        
        <div class="robot-msg" id="lumi-hint"></div>
        
        <p id="question-text" style="font-size: 1.2em; font-weight: 600; margin: 20px 0;"></p>
        
        <div id="options-box"></div>
        <div id="feedback"></div>
    </div>

    <div id="screen-final" class="hidden">
        <span class="emoji-large">🔓</span>
        <h1>SYSTÈME DÉBLOQUÉ !</h1>
        <div class="robot-msg">🤖 <strong>Lumi:</strong> "Mission accomplie ! Grâce à ton expertise linguistique, les barrières sont levées et la forêt de Vertefeuille respire à nouveau !"</div>
        <p>Tu as obtenu le grade de : <strong>Protecteur de la Nature</strong> 🌿</p>
        <button onclick="location.reload()">RECOMMENCER LA MISSION</button>
    </div>
</div>

<script>
    const questions = [
        { hint: "Commençons par tes projets ! (Futur Proche)", q: "Demain, je _________ (visiter) la montagne.", options: ["A) vais visiter", "B) vas visiter", "C) va visiter"], correct: 0 },
        { hint: "Le transport est important pour la planète.", q: "Nous _________ (prendre) le train écologique.", options: ["A) allez prendre", "B) allons prendre", "C) vont prendre"], correct: 1 },
        { hint: "Regarde ces fleurs magnifiques !", q: "Dans la forêt, il y a de _________ (beautiful flowers).", options: ["A) belles fleurs", "B) beaux fleurs", "C) belle fleurs"], correct: 0 },
        { hint: "Attention aux déchets !", q: "Où vas-tu jeter cette bouteille en plastique ?", options: ["A) Dans la rivière", "B) Sous un arbre", "C) Dans la poubelle de tri"], correct: 2 },
        { hint: "Les copains arrivent !", q: "Mes amis _________ (faire) une randonnée demain.", options: ["A) allons faire", "B) allez faire", "C) vont faire"], correct: 2 },
        { hint: "L'eau est précieuse.", q: "Il ne _________ (polluer) pas la rivière.", options: ["A) va pas polluer", "B) n'allons pas polluer", "C) ne va polluer pas"], correct: 0 },
        { hint: "Qu'est-ce qu'on met dans le sac à dos ?", q: "Pour boire de l'eau, j'utilise _________.", options: ["A) un gobelet jetable", "B) une gourde", "C) un sac en papier"], correct: 1 },
        { hint: "Et vous ? Quels sont vos plans ?", q: "Vous _________ (protéger) les animaux sauvages.", options: ["A) allez protéger", "B) allons protéger", "C) vont protéger"], correct: 0 },
        { hint: "Vocabulaire de la nature.", q: "Trouve l'intrus (the odd one out) :", options: ["A) La forêt", "B) L'ordinateur", "C) Le lac"], correct: 1 },
        { hint: "Dernière mission ! L'engagement.", q: "Promets-tu de protéger la Terre ?", options: ["A) Oui, c'est nul", "B) Non, jamais", "C) Oui, je vais faire attention !"], correct: 2 }
    ];

    let currentStep = 0;

    function startGame() {
        document.getElementById('screen-start').classList.add('hidden');
        document.getElementById('question-container').classList.remove('hidden');
        updateQuestion();
    }

    function updateQuestion() {
        const qData = questions[currentStep];
        
        // Mise à jour de la progression
        const progressPercent = (currentStep / questions.length) * 100;
        document.getElementById('progress-fill').style.width = progressPercent + "%";
        document.getElementById('progress-text').innerText = `Mission ${currentStep + 1} sur ${questions.length}`;
        
        // Contenu
        document.getElementById('lumi-hint').innerHTML = `🤖 <strong>Lumi:</strong> "${qData.hint}"`;
        document.getElementById('question-text').innerText = qData.q;
        
        const optionsBox = document.getElementById('options-box');
        optionsBox.innerHTML = '';
        
        qData.options.forEach((opt, i) => {
            const btn = document.createElement('button');
            btn.innerText = opt;
            btn.onclick = () => checkAnswer(i);
            optionsBox.appendChild(btn);
        });
        
        const fb = document.getElementById('feedback');
        fb.innerText = '';
        fb.className = '';
    }

    function checkAnswer(selected) {
        const fb = document.getElementById('feedback');
        const box = document.getElementById('game-box');
        
        if (selected === questions[currentStep].correct) {
            fb.innerText = "✅ Bravo ! Accès au verrou suivant...";
            fb.className = "correct-anim";
            
            setTimeout(() => {
                currentStep++;
                if (currentStep < questions.length) {
                    updateQuestion();
                } else {
                    showFinal();
                }
            }, 1000);
        } else {
            fb.innerText = "❌ Erreur système ! Réessaie.";
            fb.className = "wrong-anim";
            // Effet visuel d'erreur
            box.classList.add('shake');
            setTimeout(() => box.classList.remove('shake'), 400);
        }
    }

    function showFinal() {
        document.getElementById('progress-fill').style.width = "100%";
        document.getElementById('question-container').classList.add('hidden');
        document.getElementById('screen-final').classList.remove('hidden');
    }
</script>

</body>
</html># escape-game
