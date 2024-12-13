<template>
    <div class="car-game" ref="carGame" tabindex="0" @keyup="handleKeyup" @keydown="handleKeydown">
        <div class="car-game__car" ref="car" :style="computeCarStyle">
            <div class="car-game__car__wheel" :style="computeWheelsStyle"></div>
            <div class="car-game__car__wheel"></div>
            <div class="car-game__car__wheel" :style="computeWheelsStyle"></div>
            <div class="car-game__car__wheel"></div>
            <div class="car-game__car__window"></div>
            <div class="car-game__car__seat"></div>
            <div class="car-game__car__helmet"></div>
        </div>

        <div class="car-game__wall" v-for="(obstacle, index) in obstacles" :key="index"
            :style="computeObstacleStyle(index)"></div>
    </div>
</template>

<script>
const obstacles = [
    {
        x: 250,
        y: 250,
        size: 50,
    },
    {
        x: 500,
        y: 500,
        size: 50,
    },
];
export default {
    name: 'CarGame',
    data() {
        return {
            fieldX: 0,
            fieldY: 0,
            fieldSize: 0,
            obstacles: obstacles,
            x: 0,
            y: 0,
            angle: 90,
            velocityX: 0,
            velocityY: 0,
            speedY: 0,
            pressedKeys: new Set(),
            interval: null,
        };
    },
    mounted() {
        const acceleration = 0.26;
        const rotationSpeed = 2.3;
        const frictionCoefficient = 0.21;

        const fieldStats = this.$refs.carGame.getBoundingClientRect();

        this.fieldX = fieldStats.left;
        this.fieldY = fieldStats.top;
        this.fieldSize = fieldStats.width;

        this.x = this.fieldX + 100;
        this.y = this.fieldY + 100;

        this.interval = setInterval(() => {
            if (
                (this.pressedKeys.has('a') && this.pressedKeys.has('w')) ||
                (this.pressedKeys.has('d') && this.pressedKeys.has('s'))
            ) {
                this.angle -= rotationSpeed;
            } else if (
                (this.pressedKeys.has('a') && this.pressedKeys.has('s')) ||
                (this.pressedKeys.has('d') && this.pressedKeys.has('w'))
            ) {
                this.angle += rotationSpeed;
            }

            // меняется не скорость, а вектор ускорения
            let accelerationX = 0,
                accelerationY = 0;

            if (this.pressedKeys.has('w')) {
                accelerationX = acceleration * Math.sin((Math.PI * this.angle) / 180);
                accelerationY = -acceleration * Math.cos((Math.PI * this.angle) / 180);
            } else if (this.pressedKeys.has('s')) {
                accelerationX = -acceleration * Math.sin((Math.PI * this.angle) / 180);
                accelerationY = acceleration * Math.cos((Math.PI * this.angle) / 180);
            }

            // сила торможения стремится снизить скорость в 0
            let speedAngle = (Math.sign(this.velocityY) * Math.PI) / 2;

            if (Math.abs(this.velocityX) > 1e-5) {
                speedAngle = Math.atan(-this.velocityY / this.velocityX);
            }

            const frictionAngle = Math.PI - speedAngle;

            const frictionEffectX =
                frictionCoefficient * Math.cos(frictionAngle);
            const frictionEffectY =
                frictionCoefficient * Math.sin(frictionAngle);

            this.velocityX =
                Math.sign(this.velocityX) *
                Math.max(
                    0,
                    Math.abs(this.velocityX) - Math.abs(frictionEffectX)
                );
            this.velocityY =
                Math.sign(this.velocityY) *
                Math.max(
                    0,
                    Math.abs(this.velocityY) - Math.abs(frictionEffectY)
                );

            this.velocityX += accelerationX;
            this.velocityY += accelerationY;

            this.x += this.velocityX;
            this.y += this.velocityY;

            const carStats = this.$refs.car.getBoundingClientRect();

            if (
                carStats.top < this.fieldY ||
                carStats.top + carStats.height >
                this.fieldY + this.fieldSize ||
                carStats.left < this.fieldX ||
                carStats.left + carStats.width >
                this.fieldX + this.fieldSize ||
                this.checkCollision()
            ) {
                this.x = this.fieldX + 100;
                this.y = this.fieldY + 100;
                this.angle = 180;
                this.velocityX = 0;
                this.velocityY = 0;
            }
        }, 10);

        this.$refs.carGame.focus();
    },
    unmounted() {
        clearInterval(this.interval);
        this.interval = null;
    },
    computed: {
        computeCarStyle() {
            return {
                top: this.y + 'px',
                left: this.x + 'px',
                transform: `rotate(${this.angle}deg)`,
            };
        },
        computeWheelsStyle() {
            if (this.pressedKeys.has('a')) {
                return {
                    transform: `rotate(-10deg)`,
                };
            } else if (this.pressedKeys.has('d')) {
                return {
                    transform: `rotate(10deg)`,
                };
            } else {
                return {
                    transform: `rotate(0deg)`,
                };
            }
        },
    },
    methods: {
        computeObstacleStyle(index) {
            const thisObstacle = this.obstacles[index];
            return {
                width: thisObstacle.size + 'px',
                height: thisObstacle.size + 'px',
                top: thisObstacle.y + 'px',
                left: thisObstacle.x + 'px',
            };
        },
        checkCollision() {
            const cordsOfCar = this.calcCarCordsByAngle(
                (this.angle * Math.PI) / 180
            );

            for (const key in cordsOfCar) {
                const elem = cordsOfCar[key];

                for (const obstacle of this.obstacles) {
                    if (
                        obstacle.x <= this.x + 25 + elem[0] &&
                        this.x + 25 + elem[0] <= obstacle.x + obstacle.size &&
                        obstacle.y <= this.y + 50 + elem[1] &&
                        this.y + 50 + elem[1] <= obstacle.y + obstacle.size
                    ) {
                        return true;
                    }
                }
            }

            return false;
        },
        calcCarCordsByAngle(angle) {
            return {
                leftTop: this.multiplyVectorToRotationMatrix(angle, [-25, -50]),
                rightTop: this.multiplyVectorToRotationMatrix(angle, [25, -50]),
                leftBottom: this.multiplyVectorToRotationMatrix(
                    angle,
                    [-25, 50]
                ),
                rightBottom: this.multiplyVectorToRotationMatrix(
                    angle,
                    [25, 50]
                ),
                leftMiddle: this.multiplyVectorToRotationMatrix(
                    angle,
                    [-25, 0]
                ),
                rightMiddle: this.multiplyVectorToRotationMatrix(
                    angle,
                    [25, 0]
                ),
                topMiddle: this.multiplyVectorToRotationMatrix(angle, [0, -50]),
                bottomMiddle: this.multiplyVectorToRotationMatrix(
                    angle,
                    [0, 50]
                ),
            };
        },
        multiplyVectorToRotationMatrix(angle, vector) {
            return [
                vector[0] * Math.cos(angle) - vector[1] * Math.sin(angle),
                vector[0] * Math.sin(angle) + vector[1] * Math.cos(angle),
            ];
        },
        handleKeydown(e) {
            if (['w', 'a', 's', 'd', 'ц', 'ф', 'ы', 'в'].includes(e.key)) {
                this.pressedKeys = new Set(
                    this.pressedKeys.add(this.convertKey(e.key))
                );
            }
        },
        handleKeyup(e) {
            this.pressedKeys.delete(this.convertKey(e.key));
            this.pressedKeys = new Set(this.pressedKeys);
        },
        convertKey(key) {
            switch (key) {
                case 'ц':
                    return 'w';
                case 'ф':
                    return 'a';
                case 'ы':
                    return 's';
                case 'в':
                    return 'd';
                default:
                    return key;
            }
        },
    },
};
</script>

<style scoped lang="less">
.car-game {
    position: relative;
    width: 800px;
    height: 800px;
    background-color: rgb(42, 41, 34);

    &__car {
        position: absolute;
        width: 50px;
        height: 100px;
        background-color: brown;
        border-radius: 20px 20px 10px 10px;

        &__window {
            position: absolute;
            top: 26px;
            left: 12px;
            border-radius: 10px 10px 0px 0px;
            width: 26px;
            height: 20px;
            background-color: aquamarine;
        }

        &__seat {
            position: absolute;
            top: 46px;
            left: 12px;
            border-radius: 0px 0px 10px 10px;
            width: 26px;
            height: 35px;
            background-color: burlywood;
        }

        &__helmet {
            position: absolute;
            top: 55px;
            left: 17px;
            border-radius: 50%;
            width: 16px;
            height: 16px;
            background-color: black;
        }

        &__wheel {
            position: absolute;
            width: 7px;
            height: 15px;
            border-radius: 1px;
            background-color: beige;

            &:nth-of-type(odd) {
                top: 20px;
            }

            &:nth-of-type(even) {
                bottom: 15px;
            }

            &:nth-of-type(1),
            &:nth-of-type(2) {
                left: 1px;
            }

            &:nth-of-type(3),
            &:nth-of-type(4) {
                right: 1px;
            }
        }
    }

    &__wall {
        position: absolute;
        background-color: black;
    }
}
</style>
