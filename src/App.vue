<template>
  test
  <canvas ref="canvas" @click="onClick"></canvas>
</template>

<script>
import { ref, onMounted } from 'vue';

export default {
  name: 'App',
  setup() {
    const canvas = ref();
    let context;
    let nextDigTime = null;
    let stack = [
      {
        x: 10,
        y: 10,
      },
    ];
    const gridSize = 20;
    let visited = [];
    let gridRender = [];

    const shuffle = (a) => {
      var j, x, i;

      for (i = a.length - 1; i > 0; i--) {
        j = Math.floor(Math.random() * (i + 1));
        x = a[i];
        a[i] = a[j];
        a[j] = x;
      }

      return a;
    };

    const update = () => {
      if (
        (nextDigTime === null || Date.now() > nextDigTime) &&
        stack.length > 0
      ) {
        nextDigTime = Date.now();

        const directions = shuffle(['left', 'top', 'right', 'bottom']);
        const currentPosition = stack[stack.length - 1];
        let newX, newY;

        for (const direction of directions) {
          switch (direction) {
            case 'left':
              newX = currentPosition.x - 1;

              if (
                gridRender[newX] &&
                gridRender[newX][currentPosition.y] &&
                !visited.includes(gridRender[newX][currentPosition.y])
              ) {
                visited.push(gridRender[newX][currentPosition.y]);
                gridRender[currentPosition.x][currentPosition.y].left = false;
                gridRender[newX][currentPosition.y].right = false;

                stack.push({
                  x: newX,
                  y: currentPosition.y,
                });
                return;
              }
              break;

            case 'right':
              newX = currentPosition.x + 1;

              if (
                gridRender[newX] &&
                gridRender[newX][currentPosition.y] &&
                !visited.includes(gridRender[newX][currentPosition.y])
              ) {
                visited.push(gridRender[newX][currentPosition.y]);
                gridRender[currentPosition.x][currentPosition.y].right = false;
                gridRender[newX][currentPosition.y].left = false;

                stack.push({
                  x: newX,
                  y: currentPosition.y,
                });
                return;
              }
              break;

            case 'top':
              newY = currentPosition.y - 1;

              if (
                gridRender[currentPosition.x] &&
                gridRender[currentPosition.x][newY] &&
                !visited.includes(gridRender[currentPosition.x][newY])
              ) {
                visited.push(gridRender[currentPosition.x][newY]);
                gridRender[currentPosition.x][currentPosition.y].top = false;
                gridRender[currentPosition.x][newY].bottom = false;

                stack.push({
                  x: currentPosition.x,
                  y: newY,
                });
                return;
              }
              break;

            case 'bottom':
              newY = currentPosition.y + 1;

              if (
                gridRender[currentPosition.x] &&
                gridRender[currentPosition.x][newY] &&
                !visited.includes(gridRender[currentPosition.x][newY])
              ) {
                visited.push(gridRender[currentPosition.x][newY]);
                gridRender[currentPosition.x][currentPosition.y].bottom = false;
                gridRender[currentPosition.x][newY].top = false;

                stack.push({
                  x: currentPosition.x,
                  y: newY,
                });
                return;
              }
              break;
          }
        }

        stack.pop();
      }
    };

    const render = () => {
      context.fillStyle = '#fff';
      context.fillRect(0, 0, canvas.value.width, canvas.value.height);

      context.fillStyle = '#000';

      for (const x in gridRender) {
        for (const y in gridRender[x]) {
          if (gridRender[x][y].left) {
            context.fillRect(x * gridSize, y * gridSize, 2, gridSize);
          }

          if (gridRender[x][y].right) {
            context.fillRect(
              x * gridSize + (gridSize - 2),
              y * gridSize,
              2,
              gridSize
            );
          }

          if (gridRender[x][y].top) {
            context.fillRect(x * gridSize, y * gridSize, gridSize, 2);
          }

          if (gridRender[x][y].bottom) {
            context.fillRect(
              x * gridSize,
              y * gridSize + (gridSize - 2),
              gridSize,
              2
            );
          }
        }
      }

      context.fillStyle = 'red';

      for (const stackItem of stack) {
        context.fillRect(
          stackItem.x * gridSize + 4,
          stackItem.y * gridSize + 4,
          gridSize - 8,
          gridSize - 8
        );
      }
    };

    const frame = () => {
      requestAnimationFrame(frame);

      update();
      render();
    };

    const resize = () => {
      nextDigTime = null;
      visited = [];
      canvas.value.width = canvas.value.offsetWidth;
      canvas.value.height = canvas.value.offsetHeight;

      const itemsX = Math.floor(canvas.value.width / gridSize);
      const itemsY = Math.floor(canvas.value.height / gridSize);

      gridRender = [];

      for (let x = 0; x < itemsX; x++) {
        for (let y = 0; y < itemsY; y++) {
          gridRender[x] = gridRender[x] ?? [];
          gridRender[x][y] = {
            top: true,
            right: true,
            bottom: true,
            left: true,
          };
        }
      }
    };

    const onClick = (e) => {
      const gridX = Math.floor(e.pageX / gridSize);
      const gridY = Math.floor(e.pageY / gridSize);

      stack = [
        {
          x: gridX,
          y: gridY,
        },
      ];

      resize();
    };

    onMounted(() => {
      context = canvas.value.getContext('2d');
      resize();
      frame();
    });

    window.addEventListener('resize', resize);

    return { canvas, onClick };
  },
};
</script>

<style>
canvas {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
}
</style>
