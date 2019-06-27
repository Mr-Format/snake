# snake
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>贪吃蛇-灯芯</title>
    <meta name="keywords" content="贪吃蛇，前端，vue">
    <meta name="description" content="简单的益智小游戏">
    <link rel="shoutcut icon" href="./lib/ico.jpg">
    <link rel="stylesheet" href="./lib/main.css">
    <script src="./lib/vue.min.js"></script>

</head>

<body>
    <div id="app" v-cloak>
        <div class="header">
            灯芯前端作品
        </div>
        <div class="main">
            <div class="mask" v-show="isDead">
                <div class="mask-info">
                    <div class="mask-info-over">游戏结束</div>
                    <p v-if="congraduation">{{ congraduation }}</p>
                    <button class="mask-info-btn" @click="reload">再来一局</button>
                </div>
            </div>
            <div class="container">
                <div class="area-row" v-for="itemI in row" :style="'top:'+(itemI-1)*size + 'px'">
                </div>
                <div class="area-col" v-for="itemJ in col" :style="'left:'+(itemJ-1)*size + 'px'">
                </div>
                <div v-for="(item,i) in snake" :key="i" class="area-snake-body" :style="snake[i].s"></div>
                <div v-for="(item,i) in food" :key="i" class="area-food" :style="food[i].s"></div>
            </div>
        </div>
        <div class="game-footer">
            <div class="game-info">
                <span>得分：<span>{{score}}</span></span>
                <span>等级：<span>{{lev}}</span></span>
            </div>
            <button class="game-start" @click="run">启动</button>
            <button class="game-pause" @click="pause">暂停</button>
            <button class="game-reload" @click="reload">重新开始</button>
            <p>点击启动，使用上、下、左、右进行操作</p>
        </div>
    </div>
    <script>
        var vm = new Vue({
            el: '#app',
            data: {
                row: '',
                col: '',
                size: '',
                i_x: '',
                i_y: '',
                score: '',
                snake: [{
                    x: 10,
                    y: 10,
                    s: {
                        backgroundColor: "#F92929 ",
                        top: '',
                        left: ''
                    }
                }, {
                    x: 9,
                    y: 10,
                    s: {
                        backgroundColor: "#0E82F8 ",
                        top: '',
                        left: ''
                    }
                }, {
                    x: 8,
                    y: 10,
                    s: {
                        backgroundColor: "#0E82F8 ",
                        top: '',
                        left: ''
                    }
                }],
                _x: '',
                _y: '',
                _s: {},
                tail: {},
                directFlag: '1',
                food: [],
                hasFood: false,
                timer: '',
                direct: 1,
                isDead: false,
                speed: 600,
                lev: 1,
                congraduation: ''
            },
            computed: {},
            created() {
                this._initMap()
                this._initSnake()
                this._initFood(10)
            },
            mounted() {},
            updated() {},
            methods: {
                run() {
                    this.change()
                    this.timer = setInterval(() => {
                        this.move(this.direct)
                    }, this.speed)
                },
                pause() {
                    clearInterval(this.timer)
                },
                change() {
                    document.onkeydown = (e) => {
                        switch (e.keyCode) {
                            case 38:
                                this.direct = 0;
                                break;
                            case 39:
                                this.direct = 1;
                                break;
                            case 40:
                                this.direct = 2;
                                break;
                            case 37:
                                this.direct = 3;
                                break;
                            default:
                        }
                    };
                },
                move(direct) {
                    if (Math.abs(this.directFlag - direct) === 2) {
                        direct = this.directFlag
                    } else {
                        this.directFlag = direct
                    }
                    let len = this.snake.length
                    this._x = this.snake[len - 1].x
                    this._y = this.snake[len - 1].y
                    for (let i = len - 1; i > 0; i--) {
                        this.snake[i].x = this.snake[i - 1].x
                        this.snake[i].y = this.snake[i - 1].y
                    }
                    switch (direct) {
                        case 0:
                            this.snake[0].y -= 1
                            break
                        case 1:
                            this.snake[0].x += 1
                            break
                        case 2:
                            this.snake[0].y += 1
                            break
                        case 3:
                            this.snake[0].x -= 1
                        default:
                    }
                    this.eat()
                    this.dead()
                    if (this.isDead) {
                        return
                    }
                    this._initSnake()
                },
                eat() {
                    for (let i = 0; i < this.food.length; i++) {
                        if (this.snake[0].x === this.food[i].x && this.snake[0].y === this.food[i].y) {
                            this.score++;
                            this.snake.push({
                                x: this._x,
                                y: this._y,
                                s: {
                                    backgroundColor: "#0E82F8 "
                                }
                            })
                            this.tailFlag = -1
                            this._initFood(1)
                            this.food.splice(i, 1)
                            if (this.score % 10 === 0) {
                                this.speed -= 50
                                this.lev++
                                    clearInterval(this.timer)
                                this.run()
                            }
                            return
                        }
                    }
                },
                dead() {
                    if (this.snake[0].x < 0 || this.snake[0].x >= this.row || this.snake[0].y < 0 || this.snake[0].y >= this.col) {
                        this.isDead = true
                        clearInterval(this.timer)
                        this.handleStorage()
                        return
                    }
                    for (let i = 1; i < this.snake.length; i++) {
                        if (this.snake[0].x === this.snake[i].x && this.snake[0].y === this.snake[i].y) {
                            this.isDead = true
                            clearInterval(this.timer)
                            this.handleStorage()
                            return
                        }
                    }
                },
                handleStorage() {
                    var lastScore = localStorage.getItem('snake-score')
                    if (!lastScore) {
                        localStorage.setItem('snake-score', this.score)
                        return
                    } else if (this.score > lastScore) {
                        this.congraduation = "恭喜你，创造了新的记录：" + this.score + "分！"
                        localStorage.setItem('snake-score', this.score)
                        return
                    }
                },
                reload() {
                    location.reload()
                },
                _initSnake() {
                    for (let i = 0; i < this.snake.length; i++) {
                        this.snake[i].s.left = (this.snake[i].x + this.i_x) * this.size + 1 + 'px'
                        this.snake[i].s.top = (this.snake[i].y + this.i_y) * this.size + 1 + 'px'
                    }
                },
                _initFood(n) {
                    let len = this.food.length
                    for (let i = len; i < n + len; i++) {
                        this.food[i] = {}
                        let x = this.getRandom(this.col)
                        let y = this.getRandom(this.row)
                        this.checkFood(x, y)
                        if (this.hasFood) {
                            i--
                            this.hasFood = false
                            continue
                        } else {
                            this.food[i].x = x
                            this.food[i].y = y
                            this.food[i].s = {
                                left: this.food[i].x * this.size + 1 + 'px',
                                top: this.food[i].y * this.size + 1 + 'px'
                            }
                        }
                    }
                },
                checkFood(x, y) {
                    for (let i = 0; i < this.food.length; i++) {
                        if (x === this.food[i].x && y === this.food[i].y) {
                            this.hasFood = true
                            return
                        }
                    }
                    for (let i = 0; i < this.snake.length; i++) {
                        if (x === this.snake[i].x && y === this.snake[i].y) {
                            this.hasFood = true
                            return
                        }
                    }
                },
                getRandom(max) {
                    return Math.floor(Math.random() * max)
                },
                _initMap() {
                    this.row = 20
                    this.col = 20
                    this.size = 16
                    this.i_x = 0
                    this.i_y = 0
                    this.score = 0
                },
            },
        })
    </script>
</body>

</html>
```