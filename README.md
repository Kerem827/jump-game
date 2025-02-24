npm install pm2 -g
pm2 start index.html
pm2 save

<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Zıplama Oyunu</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            overflow: hidden;
            height: 100%;
            background-color: #87CEEB;
        }
        #gameCanvas {
            display: block;
            margin: 0 auto;
            background-color: #f5f5f5;
        }
        .player {
            width: 50px;
            height: 50px;
            position: absolute;
            bottom: 0;
            left: 50px;
            transition: all 0.1s ease;
        }
        .obstacle {
            width: 30px;
            height: 50px;
            background-color: #228B22;
            position: absolute;
            bottom: 0;
        }
        #score {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 30px;
            color: black;
            font-weight: bold;
        }
        #restartButton {
            position: absolute;
            top: 100px;
            left: 50%;
            transform: translateX(-50%);
            padding: 10px 20px;
            background-color: #ff6347;
            color: white;
            font-size: 20px;
            cursor: pointer;
            display: none;
        }
        #characterSelect {
            position: absolute;
            top: 150px;
            left: 50%;
            transform: translateX(-50%);
            padding: 10px;
            font-size: 18px;
            color: black;
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="800" height="400"></canvas>
<div id="score">Puan: 0</div>
<button id="restartButton" onclick="restartGame()">Tekrar Başla</button>
<div id="characterSelect">
    <label for="characters">Yeni karakter seç: </label>
    <select id="characters">
        <option value="red">Kırmızı</option>
        <option value="blue">Mavi</option>
        <option value="green">Yeşil</option>
        <option value="yellow">Sarı</option>
        <option value="purple">Mor</option>
        <option value="orange">Turuncu</option>
        <option value="pink">Pembe</option>
        <option value="brown">Kahverengi</option>
        <option value="black">Siyah</option>
        <option value="white">Beyaz</option>
    </select>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    let player = {
        x: 50,
        y: canvas.height - 50,
        width: 50,
        height: 50,
        speed: 0,
        gravity: 0.5,  // Yavaş yere düşme
        jumpPower: -15,  // Yüksek zıplama
        jumping: false,
        color: '#ff6347'  // Başlangıç rengi
    };

    let obstacles = [];
    let score = 0;
    let gameOver = false;

    // Kullanıcı tuşa basarsa zıpla
    document.addEventListener('keydown', function (e) {
        if (e.key === ' ' && !player.jumping && !gameOver) { // Space tuşu
            player.speed = player.jumpPower;
            player.jumping = true;
        }
    });

    // Engelleri oluştur
    function createObstacle() {
        const height = Math.random() * 50 + 30;
        const obstacle = {
            x: canvas.width,
            y: canvas.height - height,
            width: 30,
            height: height,
            speed: 3,  // Düşük hız
        };
        obstacles.push(obstacle);
    }

    // Engelleri hareket ettir
    function moveObstacles() {
        obstacles.forEach(obstacle => {
            obstacle.x -= obstacle.speed;
        });

        // Engelleri temizle
        obstacles = obstacles.filter(obstacle => obstacle.x + obstacle.width > 0);
    }

    // Oyun alanını güncelle
    function update() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Zıplama
        if (player.jumping) {
            player.y += player.speed;
            player.speed += player.gravity;

            if (player.y >= canvas.height - player.height) {
                player.y = canvas.height - player.height;
                player.jumping = false;
            }
        }

        // Engelleri çiz
        obstacles.forEach(obstacle => {
            ctx.fillStyle = '#228B22';
            ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
        });

        // Oyuncu karakterini çiz
        ctx.fillStyle = player.color;
        ctx.fillRect(player.x, player.y, player.width, player.height);

        // Puanı güncelle
        ctx.fillStyle = 'black';
        ctx.font = '30px Arial';
        ctx.fillText('Puan: ' + score, 20, 40);

        // Çarpma kontrolü
        obstacles.forEach(obstacle => {
            if (player.x + player.width > obstacle.x && player.x < obstacle.x + obstacle.width &&
                player.y + player.height > obstacle.y) {
                gameOver = true;
                ctx.fillStyle = 'red';
                ctx.font = '40px Arial';
                ctx.fillText('Oyun Bitti!', canvas.width / 2 - 100, canvas.height / 2);
                document.getElementById('restartButton').style.display = 'block'; // Yeniden başlatma butonunu göster
            }
        });

        // Puan artırma
        if (!gameOver) {
            score++;
        }

        if (!gameOver) {
            requestAnimationFrame(update);
        }
    }

    // Engelleri oluştur ve hareket ettir
    function gameLoop() {
        if (Math.random() < 0.005) { // Engellerin arasındaki mesafeyi daha da açtım
            createObstacle();
        }

        moveObstacles();

        if (!gameOver) {
            requestAnimationFrame(gameLoop);
        }
    }

    // Yeni karakter seçimi
    document.getElementById('characters').addEventListener('change', function () {
        const selectedColor = this.value;
        player.color = selectedColor;  // Yeni renk seçimi
    });

    // Yeniden başlatma
    function restartGame() {
        player.y = canvas.height - 50;
        player.speed = 0;
        player.jumping = false;
        score = 0;
        gameOver = false;
        obstacles = [];
        document.getElementById('restartButton').style.display = 'none';
        update();
        gameLoop();
    }

    // Başlat
    update();
    gameLoop();
</script>

</body>
</html>
