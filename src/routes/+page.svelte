<script lang="ts">
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { translations, labels } from '$lib/translations.ts';
  import "../styles/global.css";

  let svg: SVGSVGElement;
  let currentHour: number;
  let currentMinute: number;
  let currentSecond: number;
  let suggestion: string;
  let currentLanguage = 'en'; // Язык по умолчанию - английский

  // Функция для получения текущего времени
  const getCurrentTime = () => {
    const now = new Date();
    currentHour = now.getHours();
    currentMinute = now.getMinutes();
    currentSecond = now.getSeconds();
  };

  // Обновляем текущее время и отображаем правильную рекомендацию
  const highlightCurrentTime = () => {
    getCurrentTime();
    suggestion = translations[currentLanguage][currentHour];
    drawClock();
  };

  // Функция для изменения языка
  const changeLanguage = (lang) => {
    currentLanguage = lang;
    highlightCurrentTime();
  };

  // Вызов функции при монтировании компонента
  onMount(() => {
    highlightCurrentTime();
    setInterval(highlightCurrentTime, 1000); // Обновляем каждую секунду
  });

  function drawClock() {
    const width = 400;
    const height = 400;
    const radius = Math.min(width, height) / 2;

    d3.select(svg).selectAll("*").remove(); // Очистим перед новым рисованием

    const svgElem = d3.select(svg)
      .attr("width", width)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${width / 2}, ${height / 2})`);

    const hourArc = d3.arc()
      .innerRadius(100)
      .outerRadius(radius);

    const hours = d3.range(0, 24);

    // Рисуем сектора для каждого часа
    svgElem.selectAll(".hour")
      .data(hours)
      .enter()
      .append("path")
      .attr("class", "hour")
      .attr("d", d => hourArc({
        startAngle: (d * Math.PI / 12) - (Math.PI / 24),
        endAngle: ((d + 1) * Math.PI / 12) - (Math.PI / 24)
      }))
      .attr("fill", (d) => {
        if (d === currentHour) return "#ff6f61";
        if (d >= 23 || d < 8) return "#FFB6C1"; 
        return "#eaeaea";
      })
      .attr("stroke", "white")
      .style("stroke-width", "2px");

    const secondAngle = (currentSecond / 60) * 360;
    const minuteAngle = (currentMinute / 60) * 360;
    const hourAngle = ((currentHour % 24) / 24) * 360 + (currentMinute / 60) * 30;


    // Добавляем текст для каждого часа
        svgElem.selectAll(".hour-label")
          .data(hours)
          .enter()
          .append("text")
          .attr("class", "hour-label")

     .attr("x", d => (radius - 40) * Math.sin(d * Math.PI / 12))

     .attr("y", d => -(radius - 40) * Math.cos(d * Math.PI / 12))

     .attr("text-anchor", "middle")

     .attr("alignment-baseline", "middle")

     .text(d => d === 0 ? 24 : d)

     .style("font-family", "'Poppins', sans-serif")

     .style("font-size", "14px")

     .style("fill", "#333");

    // Часовая стрелка с острым концом
    svgElem.append("polygon")
      .attr("points", `-3,0 3,0 0,${-(radius - 80)}`)
      .attr("fill", "#333")
      .attr("transform", `rotate(${hourAngle})`);

    // Минутная стрелка с острым концом
    svgElem.append("polygon")
      .attr("points", `-4,0 4,0 0,${-(radius - 50)}`)
      .attr("fill", "#333")
      .attr("transform", `rotate(${minuteAngle})`);

    // Секундная стрелка с острым концом
    svgElem.append("polygon")
      .attr("points", `-1,0 1,0 0,${-(radius - 30)}`)
      .attr("fill", "#ff6f61")
      .attr("transform", `rotate(${secondAngle})`);

    const timeGroup = svgElem.append("g")
      .attr("class", "time-group");

    // Овал
    timeGroup.append("ellipse")
      .attr("cx", 0)
      .attr("cy", 0)
      .attr("rx", 55)
      .attr("ry", 55)
      .attr("fill", "white")
      .attr("stroke", "#e2e2e2")
      .attr("stroke-width", 0.5);

   timeGroup.append("text")
  .attr("class", "current-time")
  .attr("x", 0)
  .attr("y", 12)
  .attr("text-anchor", "middle")
  .text(`${String(currentHour).padStart(2, '0')}:${String(currentMinute).padStart(2, '0')}`);
  }
</script>

<div class="container">
  <div class="left-content">
    <label for="language" class="language-label">Choose Language:</label>
    <select id="language" on:change="{(e) => changeLanguage(e.target.value)}" class="language-select">
      <option value="en">English</option>
      <option value="jp">日本語 (Japanese)</option>
      <option value="ua">Українська (Ukrainian)</option>
      <option value="ru">Русский (Russian)</option>
    </select>

    <div class="recommendation">
            <h3>{labels[currentLanguage].tasksLabel}</h3>

      {#each translations[currentLanguage] as task, index}
        <p class:current-task={index === currentHour}>{task}</p>
      {/each}
    </div>
  </div>

  <div class="clock-container">
    <svg bind:this={svg}></svg>
  </div>
</div>
