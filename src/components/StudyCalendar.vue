<script setup>
import { computed } from "vue";

const props = defineProps({
  studyPlan: {
    type: Object,
    required: true,
  },
});

const getDayColor = (day) => {
  const colors = {
    monday: "#FF6B6B",
    tuesday: "#4ECDC4",
    wednesday: "#45B7D1",
    thursday: "#FFA07A",
    friday: "#98D8C8",
    saturday: "#F7DC6F",
    sunday: "#BB8FCE",
  };
  return colors[day] || "#667eea";
};

const getDayName = (day) => {
  const names = {
    monday: "Lunes",
    tuesday: "Martes",
    wednesday: "Miércoles",
    thursday: "Jueves",
    friday: "Viernes",
    saturday: "Sábado",
    sunday: "Domingo",
  };
  return names[day] || day;
};

const formatDate = (dateString) => {
  const date = new Date(dateString);
  return date.toLocaleDateString("es-ES", {
    weekday: "long",
    year: "numeric",
    month: "long",
    day: "numeric",
  });
};

const totalHours = computed(() => {
  return props.studyPlan.weekly_plan.reduce((total, block) => {
    if (block.start === "—" || block.end === "—") return total;

    const [startH, startM] = block.start.split(":").map(Number);
    const [endH, endM] = block.end.split(":").map(Number);

    const startMinutes = startH * 60 + startM;
    const endMinutes = endH * 60 + endM;

    return total + (endMinutes - startMinutes) / 60;
  }, 0);
});
</script>

<template>
  <div class="calendar-container">
    <div class="calendar-header">
      <div class="header-info">
        <h2>{{ studyPlan.subject }}</h2>
        <div class="details">
          <span>📅 Examen: {{ formatDate(studyPlan.exam_date) }}</span>
          <span>🌍 Zona: {{ studyPlan.timezone }}</span>
          <span>⏱️ Horas totales: {{ totalHours.toFixed(1) }}h</span>
        </div>
      </div>
    </div>

    <div class="study-blocks">
      <div
        v-for="(block, index) in studyPlan.weekly_plan"
        :key="index"
        class="study-card"
        :style="{ borderLeftColor: getDayColor(block.day) }"
      >
        <div class="card-header">
          <div
            class="day-badge"
            :style="{ backgroundColor: getDayColor(block.day) }"
          >
            {{ getDayName(block.day) }}
          </div>
          <div class="date-time">
            <span class="date">{{ formatDate(block.study_date) }}</span>
            <span v-if="block.start !== '—'" class="time">
              {{ block.start }} - {{ block.end }}
            </span>
            <span v-else class="exam">EXAMEN</span>
          </div>
        </div>

        <div class="card-body">
          <h3 class="topic">{{ block.topic }}</h3>

          <div class="section">
            <label>📚 Material:</label>
            <p class="material">{{ block.material_source }}</p>
          </div>

          <div class="section">
            <label>🎯 Objetivo:</label>
            <p class="goal">{{ block.goal }}</p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="studyPlan.notes && studyPlan.notes.length" class="notes-section">
      <h3>📝 Notas importantes</h3>
      <ul class="notes-list">
        <li v-for="(note, index) in studyPlan.notes" :key="index">
          {{ note }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.calendar-container {
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
}

.calendar-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 30px;
  border-radius: 12px;
  margin-bottom: 30px;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.header-info h2 {
  font-size: 2em;
  margin-bottom: 15px;
}

.details {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  font-size: 0.95em;
  opacity: 0.95;
}

.details span {
  display: flex;
  align-items: center;
  gap: 5px;
}

.study-blocks {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 30px;
}

.study-card {
  background: white;
  border-left: 5px solid #667eea;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: all 0.3s ease;
}

.study-card:hover {
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
}

.day-badge {
  display: inline-block;
  padding: 8px 16px;
  border-radius: 6px;
  color: white;
  font-weight: 600;
  font-size: 0.85em;
  text-transform: uppercase;
  min-width: 90px;
  text-align: center;
}

.date-time {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.date {
  font-size: 0.9em;
  color: #666;
  text-transform: capitalize;
}

.time {
  font-weight: 600;
  color: #333;
  font-size: 0.95em;
}

.exam {
  font-weight: 600;
  color: #e74c3c;
  font-size: 0.95em;
}

.card-body {
  padding: 20px;
}

.topic {
  font-size: 1.1em;
  color: #333;
  margin-bottom: 15px;
  font-weight: 600;
}

.section {
  margin-bottom: 15px;
}

.section label {
  display: block;
  font-weight: 600;
  color: #667eea;
  margin-bottom: 6px;
  font-size: 0.9em;
}

.material {
  color: #555;
  font-size: 0.9em;
  line-height: 1.5;
  font-style: italic;
  padding-left: 10px;
  border-left: 3px solid #e9ecef;
}

.goal {
  color: #555;
  font-size: 0.9em;
  line-height: 1.6;
  padding-left: 10px;
  border-left: 3px solid #e9ecef;
}

.notes-section {
  background: #f0f4ff;
  border: 2px solid #667eea;
  border-radius: 8px;
  padding: 20px;
  margin-top: 30px;
}

.notes-section h3 {
  color: #667eea;
  margin-bottom: 15px;
  font-size: 1.1em;
}

.notes-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.notes-list li {
  padding-left: 25px;
  position: relative;
  color: #555;
  font-size: 0.9em;
  line-height: 1.6;
}

.notes-list li:before {
  content: "✓";
  position: absolute;
  left: 0;
  color: #667eea;
  font-weight: bold;
}

@media (max-width: 640px) {
  .calendar-header {
    padding: 20px;
  }

  .header-info h2 {
    font-size: 1.5em;
  }

  .details {
    flex-direction: column;
    gap: 10px;
  }

  .card-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .day-badge {
    width: 100%;
  }
}
</style>
