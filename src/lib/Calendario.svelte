<script>
  import datos from './datos.json';
  const fechaInicio = new Date('2024-01-01');
  const fechaFin = new Date('2025-07-31');

  function getAllDates(start, end) {
    const arr = [];
    const dt = new Date(start);
    while (dt <= end) {
      arr.push(new Date(dt));
      dt.setDate(dt.getDate() + 1);
    }
    return arr;
  }

  const dias = getAllDates(fechaInicio, fechaFin);
  const dataMap = new Map();
  const detallesMap = new Map();

  // Mapear datos asegurando que guardamos toneladas, no 'via'
  for (const d of datos) {
    let fecha = d.fecha;
    if (!fecha && d.fecha_txt) {
      const [day, month, year] = d.fecha_txt.split('/');
      fecha = `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`;
    }
    if (fecha && d.toneladas != null && !dataMap.has(fecha)) {
      dataMap.set(fecha, d.toneladas);
      detallesMap.set(fecha, d);
    }
  }

  // Paleta con 7 colores para los rangos definidos
  const escalaColores = [
    '#B8FFEC',
    '#8AFEDF',
    '#2FFEC7',
    '#01f3b3',
    '#01A277',
    '#007556',
    '#006146'
  ];

  // Rangos fijos para la leyenda (toneladas)
  const rangosLeyenda = [
    { color: '#B8FFEC', start: 1, end: 1500 },
    { color: '#8AFEDF', start: 1500, end: 3000 },
    { color: '#2FFEC7', start: 3000, end: 4500 },
    { color: '#01f3b3', start: 4500, end: 6500 },
    { color: '#01A277', start: 6500, end: 10000 },
    { color: '#007556', start: 10000, end: 13000 },
    { color: '#006146', start: 13000, end: 19000 }
  ];

  function getColorPorTons(toneladas) {
    if (isNaN(toneladas) || toneladas === 0) return '#E0E0E0';
    if (toneladas <= 1500) return escalaColores[0];
    else if (toneladas <= 3000) return escalaColores[1];
    else if (toneladas <= 4500) return escalaColores[2];
    else if (toneladas <= 6500) return escalaColores[3];
    else if (toneladas <= 10000) return escalaColores[4];
    else if (toneladas <= 13000) return escalaColores[5];
    else return escalaColores[6];
  }

  function getRectColor(fecha) {
    const d = detallesMap.get(fecha);
    if (!d || d.toneladas == null) return '#E0E0E0';
    return getColorPorTons(Number(d.toneladas));
  }

  function formatTonsEU(n) {
    if (n == null || isNaN(n)) return "";
    const x = Math.round(n);
    return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
  }

  function getTooltipContent(d) {
    let html = `<b>Fecha:</b> ${d && d.fecha_txt ? d.fecha_txt : 'No disponible'}`;
    if (!d) return html + '<br>Sin datos';
    if (d.toneladas != null) html += `<br><b>Toneladas:</b> ${formatTonsEU(d.toneladas)}`;
    return html;
  }

  const anchoRect = 24;
  const altoRect = 24;
  const sep = 30;
  const offx = 6;
  const offy = 12;
  const nFilas = 15;
  const nColumnas = Math.ceil(dias.length / nFilas);

  function toISO(dia) {
    return dia.toISOString().slice(0, 10);
  }

  const viewWidth = nColumnas * sep + offx * 2;
  const viewHeight = nFilas * sep + offy * 2;

  const mesesAbrev = ['Ene', 'Feb', 'Mar', 'Abr', 'May', 'Jun', 'Jul', 'Ago', 'Sep', 'Oct', 'Nov', 'Dic'];

  let tooltipVisible = false;
  let tooltipX = 0;
  let tooltipY = 0;
  let tooltipContent = '';
  let tooltipColor = '#000';
  let container;
</script>

<style>
  body {
    margin: 0;
  }

  svg {
    display: block;
  }

  .tooltip {
    position: absolute;
    background: white;
    border: 2px solid;
    padding: 10px;
    font-size: 0.95rem;
    pointer-events: none;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
    border-radius: 5px;
    z-index: 10;
    max-width: 240px;
    white-space: nowrap;
    text-align: left;
  }

  .leyenda {
    font-size: 0.8rem;
    font-family: Helvetica, sans-serif;
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
    align-items: center;
    gap: 8px;
    margin: 6px 0 4px 6px;
    min-height: 15px;
  }

  .leyenda-item {
    display: flex;
    align-items: center;
    gap: 3px;
    margin-right: 10px;
    white-space: nowrap;
  }

  .leyenda-color {
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 2px;
    border: 1px solid #bbb;
  }

  .month-label {
    font-family: Helvetica, sans-serif;
    font-size: 10px;
    fill: #333;
    user-select: none;
  }
@media (max-width: 600px) {
  .leyenda {
    font-size: 0.5rem;
    gap: 6px;
    margin-top: 5px;
    min-height: 8px;
    line-height: 0.1;
    margin-bottom: 10px;
  }
  .leyenda-item {
    margin-right: 6px;
    white-space: nowrap;
    line-height: 0.5;
  }
  .leyenda-color {
    width: 6px;
    height: 6px;
  }

}
</style>

<!-- Leyenda -->
<div class="leyenda" style="flex-wrap: wrap; gap: 12px;">
  <span class="leyenda-item">
    Datos diarios desde 2024 hasta el 28 de julio de 2025.
  </span>

  {#each rangosLeyenda as rango}
    <span class="leyenda-item" style="white-space: nowrap;">
      <span 
        class="leyenda-color" 
        style="background-color: {rango.color}; border-color: {rango.color};"
      ></span>
      {formatTonsEU(rango.start)} – {formatTonsEU(rango.end)} toneladas
    </span>
  {/each}
</div>

<!-- Contenedor visual -->
<div
  bind:this={container}
  style="position: relative; width: 100%; overflow-x: auto; white-space: nowrap;"
>
  <svg
    viewBox={`0 0 ${viewWidth} ${viewHeight}`}
    width={viewWidth}
    height={viewHeight}
    preserveAspectRatio="xMidYMid meet"
    style="display: inline-block; max-width: 100%; height: auto;"
  >
    {#each dias as dia, idx (idx)}
      {#if dia.getDate() === 1}
        <!-- Línea que delimita el primer día del mes -->
        <line
          x1={offx + Math.floor(idx / nFilas) * sep}
          y1={offy + (idx % nFilas) * sep - 1}
          x2={offx + Math.floor(idx / nFilas) * sep + anchoRect}
          y2={offy + (idx % nFilas) * sep - 1}
          stroke="#000"
          stroke-width="1.5"
        />
        <!-- Abreviatura del mes, más arriba -->
        <text
          x={offx + Math.floor(idx / nFilas) * sep + anchoRect / 2}
          y={offy + (idx % nFilas) * sep - 5}
          class="month-label"
          text-anchor="middle"
          alignment-baseline="middle"
        >
          {mesesAbrev[dia.getMonth()]}
        </text>
      {/if}
      <rect
        x={offx + Math.floor(idx / nFilas) * sep}
        y={offy + (idx % nFilas) * sep}
        width={anchoRect}
        height={altoRect}
        fill={getRectColor(toISO(dia))}
        stroke="#fff"
        stroke-width="1"
        on:mouseenter={() => {
          const fecha = toISO(dia);
          const d = detallesMap.get(fecha);
          tooltipVisible = true;
          tooltipContent = getTooltipContent(d, fecha);
          tooltipColor = getRectColor(fecha);
        }}
        on:mouseleave={() => {
          tooltipVisible = false;
        }}
        on:mousemove={(e) => {
          const bounds = container.getBoundingClientRect();
          const tooltipWidth = 240;
          const tooltipHeight = 100;
          const padding = 10;

          let relX = e.clientX - bounds.left + padding;
          let relY = e.clientY - bounds.top + padding;

          if (relX + tooltipWidth > bounds.width) {
            relX = bounds.width - tooltipWidth - padding;
          }
          if (relY + tooltipHeight > bounds.height) {
            relY = bounds.height - tooltipHeight - padding;
          }
          if (relX < padding) relX = padding;
          if (relY < padding) relY = padding;

          tooltipX = relX;
          tooltipY = relY;
        }}
      />
    {/each}
  </svg>

  {#if tooltipVisible}
    <div
      class="tooltip"
      style="
        left: {tooltipX}px;
        top: {tooltipY}px;
        border-color: {tooltipColor};
      "
    >{@html tooltipContent}</div>
  {/if}
</div>
