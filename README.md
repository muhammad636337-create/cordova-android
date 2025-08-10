<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>لعبة تعبان - Snake Game</title>
  <style>
    body {
      background: #222;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      font-family: 'Arial', sans-serif;
      color: white;
      user-select: none;
    }

    #game {
      position: relative;
      width: 600px;
      height: 600px;
      background: #111;
      border: 3px solid #eee;
      overflow: hidden;
    }

    .cell {
      position: absolute;
      width: 20px;
      height: 20px;
      box-sizing: border-box;
    }

    /* الأكل */
    .food {
      background: white;
      border-radius: 50%;
    }

    /* الثعبان الرئيسي */
    .snake-main {
      background: limegreen;
      border-radius: 4px;
    }

    /* ذيل الثعبان الرئيسي */
    .snake-main.body {
      background: green;
    }

    /* تعابين اخرى - ألوان مختلفة */
    .snake-other {
      border-radius: 4px;
    }

    /* رؤوس تعابين اخرى بألوان مميزة */
    .snake-other.head1 {
      background: #ff6347; /* طماطم */
    }
    .snake-other.body1 {
      background: #cc4c3c;
    }

    .snake-other.head2 {
      background: #1e90ff; /* أزرق */
    }
    .snake-other.body2 {
      background: #1461b9;
    }

    /* رسالة النهاية */
    #gameOver {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 48px;
      background: rgba(0,0,0,0.8);
      padding: 20px 40px;
      border-radius: 10px;
      display: none;
      z-index: 10;
    }

  </style>
</head>
<body>
  <div id="game">
    <div id="gameOver">انتهت اللعبة! اضغط R لإعادة اللعب</div>
  </div>

  <script>
    // إعدادات اللعبة
    const gridSize = 30; // عدد خلايا العرض والارتفاع (600/20=30)
    const cellSize = 20; // حجم الخلية px
    const game = document.getElementById('game');
    const gameOverEl = document.getElementById('gameOver');

    // الاتجاهات
    const directions = {
      ArrowUp: { x: 0, y: -1 },
      ArrowDown: { x: 0, y: 1 },
      ArrowLeft: { x: -1, y: 0 },
      ArrowRight: { x: 1, y: 0 }
    };

    // الثعبان الرئيسي
    let mainSnake = {
      body: [{ x: 5, y: 5 }],
      direction: directions.ArrowRight,
      grow: 0,
    };

    // تعابين اخرى (ثعبانين كمثال)
    let otherSnakes = [
      {
        body: [{ x: 15, y: 15 }, { x: 14, y: 15 }, { x: 13, y: 15 }],
        direction: directions.ArrowRight,
        headClass: 'head1',
        bodyClass: 'body1'
      },
      {
        body: [{ x: 20, y: 10 }, { x: 20, y: 11 }, { x: 20, y: 12 }],
        direction: directions.ArrowUp,
        headClass: 'head2',
        bodyClass: 'body2'
      }
    ];

    // مكان الأكل
    let food = { x: 10, y: 10 };

    // حالة اللعبة
    let gameRunning = true;

    // رسم الخلايا
    function drawCell(x, y, className) {
      const cell = document.createElement('div');
      cell.classList.add('cell', className);
      cell.style.left = x * cellSize + 'px';
      cell.style.top = y * cellSize + 'px';
      return cell;
    }

    // توليد مكان عشوائي للأكل ولا يصادف أي ثعبان
    function generateFood() {
      while (true) {
        const x = Math.floor(Math.random() * gridSize);
        const y = Math.floor(Math.random() * gridSize);
        // تأكد انه مش على أي ثعبان
        if (
          !mainSnake.body.some(seg => seg.x === x && seg.y === y) &&
          !otherSnakes.some(snake => snake.body.some(seg => seg.x === x && seg.y === y))
        ) {
          return { x, y };
        }
      }
    }

    // تحقق من الاصطدام داخل شبكة
    function isCollision(pos1, pos2) {
      return pos1.x === pos2.x && pos1.y === pos2.y;
    }

    // تحديث الثعبان الرئيسي
    function updateMainSnake() {
      const head = { 
        x: mainSnake.body[0].x + mainSnake.direction.x, 
        y: mainSnake.body[0].y + mainSnake.direction.y 
      };

      // حدود الخريطة (لو تعدي يموت)
      if (head.x < 0 || head.x >= gridSize || head.y < 0 || head.y >= gridSize) {
        endGame();
        return;
      }

      // تحقق من الاصطدام بذيل نفسه
      if (mainSnake.body.some((seg, idx) => idx !== 0 && isCollision(head, seg))) {
        endGame();
        return;
      }

      // تحقق من الاصطدام بتعابين أخرى (أي جزء من الثعبان)
      for (const snake of otherSnakes) {
        // لو دخل رأس تعبان آخر من الخلف (يعني جسمه بدون رأس)
        for (let i = 1; i < snake.body.length; i++) {
          if (isCollision(head, snake.body[i])) {
            endGame();
            return;
          }
        }

        // لو لمس رأس تعبان آخر
        if (isCollision(head, snake.body[0])) {
          endGame();
          return;
        }
      }

      // أضف الرأس الجديد
      mainSnake.body.unshift(head);

      // تحقق إذا أكل الأكل
      if (isCollision(head, food)) {
        mainSnake.grow += 1;
        food = generateFood();
      }

      // إذا ما فيش نمو نقص آخر ذيل
      if (mainSnake.grow > 0) {
        mainSnake.grow--;
      } else {
        mainSnake.body.pop();
      }
    }

    // تحديث تعابين أخرى (تتحرك تلقائياً)
    function updateOtherSnakes() {
      otherSnakes.forEach(snake => {
        const head = snake.body[0];
        let newHead = { 
          x: head.x + snake.direction.x, 
          y: head.y + snake.direction.y 
        };

        // إذا خرج من حدود، يخلي الاتجاه عشوائي مختلف عشان ما يموتش فجأة
        if (newHead.x < 0 || newHead.x >= gridSize || newHead.y < 0 || newHead.y >= gridSize) {
          snake.direction = randomDirection();
          newHead = {
            x: head.x + snake.direction.x,
            y: head.y + snake.direction.y
          };
        }

        // أضف الرأس الجديد
        snake.body.unshift(newHead);

        // إزالة آخر جزء (لازم الثعبان يبقى بنفس الطول)
        snake.body.pop();

        // تحقق من الاصطدام مع الثعبان الرئيسي
        // لو رأس ثعبان اخر لمس رأس الثعبان الرئيسي -> تموت
        if (isCollision(newHead, mainSnake.body[0])) {
          endGame();
          return;
        }
      });
    }

    // اتجاه عشوائي من الاتجاهات الأربعة
    function randomDirection() {
      const keys = Object.keys(directions);
      return directions[keys[Math.floor(Math.random() * keys.length)]];
    }

    // رسم اللعبة
    function draw() {
      // نظف الشاشة
      game.innerHTML = '';
      game.appendChild(gameOverEl); // إعادة العنصر

      // ارسم الأكل
      const foodCell = drawCell(food.x, food.y, 'food');
      game.appendChild(foodCell);

      // ارسم الثعبان الرئيسي
      mainSnake.body.forEach((segment, idx) => {
        const className = idx === 0 ? 'snake-main' : 'snake-main body';
        const cell = drawCell(segment.x, segment.y, className);
        game.appendChild(cell);
      });

      // ارسم تعابين أخرى
      otherSnakes.forEach(snake => {
        snake.body.forEach((segment, idx) => {
          const className = idx === 0 ? `snake-other ${snake.headClass}` : `snake-other ${snake.bodyClass}`;
          const cell = drawCell(segment.x, segment.y, className);
          game.appendChild(cell);
        });
      });
    }

    // إنهاء اللعبة
    function endGame() {
      gameRunning = false;
      gameOverEl.style.display = 'block';
    }

    // إعادة تشغيل اللعبة
    function resetGame() {
      mainSnake = {
        body: [{ x: 5, y: 5 }],
        direction: directions.ArrowRight,
        grow: 0,
      };
      otherSnakes = [
        {
          body: [{ x: 15, y: 15 }, { x: 14, y: 15 }, { x: 13, y: 15 }],
          direction: directions.ArrowRight,
          headClass: 'head1',
          bodyClass: 'body1'
        },
        {
          body: [{ x: 20, y: 10 }, { x: 20, y: 11 }, { x: 20, y: 12 }],
          direction: directions.ArrowUp,
          headClass: 'head2',
          bodyClass: 'body2'
        }
      ];
      food = generateFood();
      gameRunning = true;
      gameOverEl.style.display = 'none';
    }

    // تحكمات لوحة المفاتيح
    window.addEventListener('keydown', e => {
      if (!gameRunning && (e.key === 'r' || e.key === 'R')) {
        resetGame();
        return;
      }

      if (!directions[e.key]) return;

      const newDir = directions[e.key];
      // منع الرجوع للخلف (مثلاً من اليمين للشمال مباشرة)
      const currentDir = mainSnake.direction;
      if (mainSnake.body.length > 1) {
        if (newDir.x === -currentDir.x && newDir.y === -currentDir.y) {
          return;
        }
      }
      mainSnake.direction = newDir;
    });

    // تحديث اللعبة كل 150 مللي ثانية
    function gameLoop() {
      if (!gameRunning) return;

      updateMainSnake();
      updateOtherSnakes();
      draw();

      setTimeout(gameLoop, 150);
    }

    // بداية اللعبة
    resetGame();
    gameLoop();
  </script>
</body>
</html>  