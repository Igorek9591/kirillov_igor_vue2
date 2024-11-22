<template>
    <div
        class="car-game"
        ref="carGame"
        tabindex="0"
        @keyup="handleKeyup"
        @keydown="handleKeydown"
    >
        <div class="car-game__car" ref="car" :style="computeCarStyle">
            <div class="car-game__car__wheel" :style="computeWheelsStyle"></div>
            <div class="car-game__car__wheel"></div>
            <div class="car-game__car__wheel" :style="computeWheelsStyle"></div>
            <div class="car-game__car__wheel"></div>
            <div class="car-game__car__window"></div>
            <div class="car-game__car__seat"></div>
            <div class="car-game__car__helmet"></div>
        </div>

        <div class="car-game__wall"></div>
    </div>
</template>

<script>
export default {
    name: 'CarGame',
    data() {
        return {
            x: 100,
            y: 100,
            angle: 0,
            velocityX: 0,
            velocityY: 0,
            pressedKeys: new Set(),
            interval: null,
        };
    },
    mounted() {
        const forwardSpeed = 5;
        const rotationSpeed = 2;

        this.interval = setInterval(() => {
            if (this.pressedKeys.size === 0) return;

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

            this.velocityX = 0;
            this.velocityY = 0;

            if (this.pressedKeys.has('s')) {
                this.velocityX =
                    -forwardSpeed * Math.sin((Math.PI * this.angle) / 180);
                this.velocityY =
                    forwardSpeed * Math.cos((Math.PI * this.angle) / 180);
            }

            if (this.pressedKeys.has('w')) {
                this.velocityX =
                    forwardSpeed * Math.sin((Math.PI * this.angle) / 180);
                this.velocityY =
                    -forwardSpeed * Math.cos((Math.PI * this.angle) / 180);
            }

            this.x += this.velocityX;
            this.y += this.velocityY;

            const rectStats = this.$refs.car.getBoundingClientRect();

            if (
                rectStats.top < 0 ||
                rectStats.top + rectStats.height > 800 ||
                rectStats.left < 0 ||
                rectStats.left + rectStats.width > 800 ||
                this.checkCollision()
            ) {
                this.x = 100;
                this.y = 100;
                this.angle = 0;
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
        checkCollision() {
            const cordsOfCar = this.calcCarCordsByAngle(
                (this.angle * Math.PI) / 180
            );

            for (const key in cordsOfCar) {
                const elem = cordsOfCar[key];

                if (
                    350 <= this.x + 25 + elem[0] &&
                    this.x + 25 + elem[0] <= 450 &&
                    350 <= this.y + 50 + elem[1] &&
                    this.y + 50 + elem[1] <= 450
                ) {
                    return true;
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
            };
        },
        multiplyVectorToRotationMatrix(angle, vector) {
            return [
                vector[0] * Math.cos(angle) - vector[1] * Math.sin(angle),
                vector[0] * Math.sin(angle) + vector[1] * Math.cos(angle),
            ];
        },
        handleKeydown(e) {
            this.pressedKeys = new Set(this.pressedKeys.add(e.key));
        },
        handleKeyup(e) {
            this.pressedKeys.delete(e.key);
            this.pressedKeys = new Set(this.pressedKeys);
        },
    },
};
</script>

<style scoped lang="less">
.car-game {
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
        top: 350px;
        left: 350px;
        background-color: black;
        width: 100px;
        height: 100px;
    }
}
</style>
