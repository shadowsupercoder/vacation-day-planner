<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let svg;
  let currentHour;
  let currentMinute;
  let currentSecond;
  let suggestion;

  // Массив рекомендаций для каждого часа
  const suggestions = [
    "00:00 – Сон.",
    "01:00 – Сон.",
    "02:00 – Сон.",
    "03:00 – Сон.",
    "04:00 – Сон.",
    "05:00 – Сон.",
    "06:00 – Сон.",
    "07:00 – Сон.",
    "08:00 – Завтрак и начало активного дня.",
    "09:00 – Экскурсия по местным музеям или галереям.",
    "10:00 – Утренняя прогулка на велосипеде или пешком.",
    "11:00 – Шопинг на местных рынках или ремесленных ярмарках.",
    "12:00 – Вкусный обед в местном кафе или ресторане.",
    "13:00 – Час дневного отдыха или дневной сон.",
    "14:00 – Чтение книги в парке или на пляже.",
    "15:00 – Посещение бассейна или пляжа для плавания.",
    "16:00 – Спа-процедуры или массаж.",
    "17:00 – Вечерняя прогулка по историческим местам.",
    "18:00 – Ужин в ресторане с видом на закат.",
    "19:00 – Прогулка по набережной или вечерний рынок.",
    "20:00 – Концерт живой музыки или ужин в приятной компании.",
    "21:00 – Лёгкий перекус и вечерние развлечения.",
    "22:00 – Коктейли или дегустация местных напитков.",
    "23:00 – Сон.",
  ];

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
    suggestion = suggestions[currentHour];
    drawClock();
  };

  // Функция для рисования часов
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
        // Подсветка текущего времени
        if (d === currentHour) return "#ff6f61"; // Подсветка текущего часа
        // Подсветка времени сна
        if (d >= 23 || d < 8) return "#FFB6C1"; // Время сна
        return "#eaeaea"; // Остальные часы
      })
      .attr("stroke", "white")
      .style("stroke-width", "2px");

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

    // Добавляем стрелки
    const secondAngle = (currentSecond / 60) * 360;
    const minuteAngle = (currentMinute / 60) * 360;
    const hourAngle = ((currentHour % 24) / 24) * 360 + (currentMinute / 60) * 30;

    // Часовая стрелка с острым концом, основание в центре
svgElem.append("polygon")
  .attr("points", `-3,0 3,0 0,${-(radius - 80)}`)  // Основание в центре, вершина наружу
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
  .attr("rx", 55)  // Горизонтальный радиус
  .attr("ry", 55)  // Вертикальный радиус
  .attr("fill", "white")
  .attr("stroke", "#e2e2e2")
  .attr("stroke-width", 0.5);

// Текст
timeGroup.append("text")
  .attr("class", "current-time")
  .attr("x", 0)
  .attr("y", 12)
  .attr("text-anchor", "middle")
  .style("font-family", "'Poppins', sans-serif")
  .style("font-size", "36px")
  .style("font-weight", "600")
  .style("fill", "#ff6f61")
  .text(`${String(currentHour).padStart(2, '0')}:${String(currentMinute).padStart(2, '0')}`);

  }

  // Вызов функции при монтировании компонента
  onMount(() => {
    highlightCurrentTime();
    setInterval(highlightCurrentTime, 1000); // Обновляем каждую секунду
  });
</script>

<!-- SVG для часов -->
<svg bind:this={svg}></svg>

<!-- Рекомендация -->
<div class="recommendation">

  <p class="suggestion">{suggestion}</p>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

  svg {
    background-color: white;
    border-radius: 50%;
    display: block;
    margin: 20px auto;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  }

  .recommendation {
    text-align: center;
    font-family: 'Poppins', sans-serif;
  }

  .time {
    font-size: 36px;
    font-weight: 600;
    color: #ff6f61;
    margin: 10px 0;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  }

  .suggestion {
    font-size: 18px;
    font-weight: 400;
    color: #333;
    max-width: 400px;
    margin: 0 auto;
    animation: fadeIn 1s ease-in-out;
  }

  /* Анимация плавного появления текста */
  @keyframes fadeIn {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  .hour-label {
    font-size: 14px;
    font-weight: 400;
    fill: #333;
  }

  .current-time {
    font-size: 36px;
    font-weight: 600;
    color: #ff6f61;
    margin: 10px 0;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  }
</style>
