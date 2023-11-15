<!--
/**
 * @fileoverview DependencyLines component
 * @license MIT
 * @author Rafal Pospiech <neuronet.io@gmail.com>
 * @package GanttElastic
 */
-->
<template>
  <svg
    x="0"
    y="0"
    width="100%"
    height="100%"
    class="gantt-elastic__chart-dependency-lines-container"
    :style="{ ...root.style['chart-dependency-lines-container'] }"
  >
    <defs>
      <marker id="markerArrowCritical" markerHeight="11" markerWidth="11" refX="8.5" refY="3" viewBox="0 0 9 6" orient="auto-start-reverse" markerUnits="userSpaceOnUse">
        <path d="M3,0 L3,6 L9,3 z" :style="{fill: root.style['chart-dependency-lines-critical-path']['stroke']}"></path>
      </marker>
      <marker id="markerArrow" markerHeight="11" markerWidth="11" refX="8.5" refY="3" viewBox="0 0 9 6" orient="auto-start-reverse" markerUnits="userSpaceOnUse">
        <path d="M3,0 L3,6 L9,3 z" :style="{fill: root.style['chart-dependency-lines-path']['stroke']}"></path>
      </marker>
    </defs>
    <g v-for="task in dependencyTasks" :key="task.id" :task="task">
      <path
        class="gantt-elastic__chart-dependency-lines-path"
        :style="{
          ...(root.state.options.showCritical&&dependencyLine.isCritical?root.style['chart-dependency-lines-critical-path']:root.style['chart-dependency-lines-path']),
          ...(root.state.options.showCritical&&dependencyLine.isCritical?task.style['chart-dependency-lines-critical-path']:task.style['chart-dependency-lines-path']),
          ...task.style['chart-dependency-lines-path-' + dependencyLine.task_id],
          'marker-end': root.state.options.showCritical && dependencyLine.isCritical ? 'url(#markerArrowCritical)' : 'url(#markerArrow)',
        }"
        v-for="dependencyLine in task.dependencyLines"
        :key="dependencyLine.id"
        :task="task"
        :d="dependencyLine.points"
      ></path>
    </g>
  </svg>
</template>

<script>
export default {
  name: 'DependencyLines',
  inject: ['root'],
  props: ['tasks'],
  data() {
    return {};
  },
  methods: {
    /**
     * Get path points
     *
     * @param {any} fromTaskId
     * @param {any} toTaskId
     * @returns {string}
     */
    getPoints(fromTaskId, toTaskId) {
      const fromTask = this.root.getTask(fromTaskId);
      const toTask = this.root.getTask(toTaskId);
      const state = this.root.state;
      const gap = state.options.chart.grid.horizontal.gap;
      const taskHeight = state.options.row.height;
      const rowHeight = (taskHeight + gap * 2);
      if (
        fromTask === null ||
        toTask === null ||
        !this.root.isTaskVisible(toTask) ||
        !this.root.isTaskVisible(fromTask)
      ) {
        return null;
      }
      const startX = fromTask.x + fromTask.width;
      const startY = fromTask.y + fromTask.height / 2;
      const stopX = toTask.x;
      const stopY = toTask.y + toTask.height / 2;
      const distanceX = stopX - startX;
      // eslint-disable-next-line no-unused-vars
      let distanceY;
      let yMultiplier = 1;
      if (stopY >= startY) {
        distanceY = stopY - startY;
      } else {
        distanceY = startY - stopY;
        yMultiplier = -1;
      }
      const offset = 10;
      const roundness = 4;
      let points = `M ${startX} ${startY}
          L ${startX + offset},${startY} `;
      if (state.options.useOldDependecyLine) {
        const isBefore = distanceX <= offset + roundness;
        if (isBefore) {
          points += `Q ${startX + offset + roundness},${startY} ${startX + offset + roundness},${startY +
          roundness * yMultiplier}
            L ${startX + offset + roundness},${startY + (distanceY * yMultiplier) / 2 - roundness * yMultiplier}
            Q ${startX + offset + roundness},${startY + (distanceY * yMultiplier) / 2} ${startX + offset},${startY +
          (distanceY * yMultiplier) / 2}
            L ${startX - offset + distanceX},${startY + (distanceY * yMultiplier) / 2}
            Q ${startX - offset + distanceX - roundness},${startY + (distanceY * yMultiplier) / 2} ${startX -
          offset +
          distanceX -
          roundness},${startY + (distanceY * yMultiplier) / 2 + roundness * yMultiplier}
            L ${startX - offset + distanceX - roundness},${stopY - roundness * yMultiplier}
            Q ${startX - offset + distanceX - roundness},${stopY} ${startX - offset + distanceX},${stopY}
            L ${stopX},${stopY}`;
        } else {
          points += `L ${startX + distanceX / 2 - roundness},${startY}
            Q ${startX + distanceX / 2},${startY} ${startX + distanceX / 2},${startY + roundness * yMultiplier}
            L ${startX + distanceX / 2},${stopY - roundness * yMultiplier}
            Q ${startX + distanceX / 2},${stopY} ${startX + distanceX / 2 + roundness},${stopY}
            L ${stopX},${stopY}`;
        }
      } else {
        const isBefore = distanceX < 0;
        if (isBefore) {
          points += `Q ${startX + offset + roundness},${startY} ${startX + offset + roundness},${startY +
          roundness * yMultiplier}
            L ${startX + offset + roundness},${startY + (rowHeight * yMultiplier) / 2 - roundness * yMultiplier}
            Q ${startX + offset + roundness},${startY + (rowHeight * yMultiplier) / 2} ${startX + offset},${startY +
          (rowHeight * yMultiplier) / 2}
            L ${startX - offset + distanceX},${startY + (rowHeight * yMultiplier) / 2}
            Q ${startX - offset + distanceX - roundness},${startY + (rowHeight * yMultiplier) / 2} ${startX -
          offset +
          distanceX -
          roundness},${startY + (rowHeight * yMultiplier) / 2 + roundness * yMultiplier}
            L ${startX - offset + distanceX - roundness},${stopY - roundness * yMultiplier}
            Q ${startX - offset + distanceX - roundness},${stopY} ${startX - offset + distanceX},${stopY}
            L ${stopX},${stopY}`;
        } else {
          points += `L ${startX + distanceX + offset - roundness},${startY}
            Q ${startX + distanceX + offset },${startY} ${startX + distanceX + offset },${startY + roundness * yMultiplier}
            L ${stopX+offset},${yMultiplier === 1 ? toTask.y : toTask.y + taskHeight}`;
        }
      }
      return points;
    },
  },
  computed: {
    /**
     * Get tasks which are dependent on other tasks
     *
     * @returns {array}
     */
    dependencyTasks() {
      return this.tasks
        .filter(task => typeof task.dependentOn !== 'undefined')
        .map(task => {
          task.dependencyLines = task.dependentOn.map(id => {
              const preTask = this.tasks.find((t) => t.id === id);
              return {
                points: this.getPoints(id, task.id),
                task_id: id,
                isCritical: (preTask && preTask.isCritical) || false,
              };
            })
            .sort((x, y) => Number(x.isCritical) - Number(y.isCritical));
          return task;
        })
        .filter((task) => task.dependencyLines.points !== null)
        .reverse();
    },
  },
};
</script>
