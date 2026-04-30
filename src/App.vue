<script setup>
import { ref } from "vue";
import StudyCalendar from "./components/StudyCalendar.vue";

const subject = ref("");
const examDate = ref("");
const timezone = ref("UTC");
const files = ref(null);
const freeTimeSlots = ref([
  { day: "monday", start: "18:00", end: "20:00" },
  { day: "wednesday", start: "19:00", end: "22:00" },
  { day: "saturday", start: "10:00", end: "13:00" },
]);

const loading = ref(false);
const studyPlan = ref(null);
const error = ref(null);
const totalSize = ref(0);

const timezones = [
  "UTC",
  "America/New_York",
  "America/Chicago",
  "America/Denver",
  "America/Los_Angeles",
  "Europe/London",
  "Europe/Paris",
  "Europe/Madrid",
  "Asia/Tokyo",
  "Asia/Shanghai",
  "Australia/Sydney",
];

const days = [
  "monday",
  "tuesday",
  "wednesday",
  "thursday",
  "friday",
  "saturday",
  "sunday",
];

const handleFileChange = (event) => {
  const selectedFiles = event.target.files;
  let total = 0;

  for (let file of selectedFiles) {
    total += file.size;
  }

  totalSize.value = total;

  if (total > 6 * 1024 * 1024) {
    error.value = "El tamaño total de archivos no puede exceder 6MB";
    files.value = null;
    return;
  }

  error.value = null;
  files.value = selectedFiles;
};

const addSlot = () => {
  freeTimeSlots.value.push({ day: "monday", start: "18:00", end: "20:00" });
};

const removeSlot = (index) => {
  freeTimeSlots.value.splice(index, 1);
};

const formatFileSize = (bytes) => {
  if (bytes === 0) return "0 Bytes";
  const k = 1024;
  const sizes = ["Bytes", "KB", "MB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return Math.round((bytes / Math.pow(k, i)) * 100) / 100 + " " + sizes[i];
};

const handleSubmit = async () => {
  error.value = null;

  if (
    !subject.value ||
    !examDate.value ||
    !files.value ||
    files.value.length === 0
  ) {
    error.value = "Por favor completa todos los campos";
    return;
  }

  loading.value = true;

  try {
    const formData = new FormData();
    formData.append("subject", subject.value);
    formData.append("exam_date", examDate.value);
    formData.append("timezone", timezone.value);
    formData.append("free_time_slots", JSON.stringify(freeTimeSlots.value));

    for (let file of files.value) {
      formData.append("files", file);
    }

    const response = await fetch(
      "https://u4cu7pffql2foifqyrubuqk6fe0vfxxj.lambda-url.us-east-1.on.aws/study-plan",
      {
        method: "POST",
        body: formData,
      },
    );

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(
        errorData.detail?.[0]?.msg || "Error al generar el plan de estudio",
      );
    }

    studyPlan.value = await response.json();
  } catch (err) {
    error.value = err.message || "Error al conectar con el servidor";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const reset = () => {
  subject.value = "";
  examDate.value = "";
  timezone.value = "UTC";
  files.value = null;
  studyPlan.value = null;
  error.value = null;
  totalSize.value = 0;
  freeTimeSlots.value = [
    { day: "monday", start: "18:00", end: "20:00" },
    { day: "wednesday", start: "19:00", end: "22:00" },
    { day: "saturday", start: "10:00", end: "13:00" },
  ];
};
</script>

<template>
  <div class="container">
    <header class="header">
      <h1>📚 University Study Planner</h1>
      <p>
        Carga tus materiales de estudio y obtén un plan de calendario
        personalizado
      </p>
    </header>

    <div v-if="!studyPlan" class="form-section">
      <form @submit.prevent="handleSubmit" class="form">
        <div class="form-group">
          <label for="subject">Materia</label>
          <input
            id="subject"
            v-model="subject"
            type="text"
            placeholder="Ej: Calidad de Software"
            class="input"
          />
        </div>

        <div class="form-row">
          <div class="form-group">
            <label for="examDate">Fecha del Examen</label>
            <input id="examDate" v-model="examDate" type="date" class="input" />
          </div>

          <div class="form-group">
            <label for="timezone">Zona Horaria</label>
            <select v-model="timezone" id="timezone" class="input">
              <option v-for="tz in timezones" :key="tz" :value="tz">
                {{ tz }}
              </option>
            </select>
          </div>
        </div>

        <div class="form-group">
          <label>Slots de Tiempo Libre</label>
          <div class="slots-container">
            <div
              v-for="(slot, index) in freeTimeSlots"
              :key="index"
              class="slot-row"
            >
              <select v-model="slot.day" class="input">
                <option v-for="day in days" :key="day" :value="day">
                  {{ day }}
                </option>
              </select>
              <input v-model="slot.start" type="time" class="input" />
              <span>a</span>
              <input v-model="slot.end" type="time" class="input" />
              <button
                type="button"
                @click="removeSlot(index)"
                class="btn-danger"
              >
                ✕
              </button>
            </div>
          </div>
          <button type="button" @click="addSlot" class="btn-secondary">
            + Agregar slot
          </button>
        </div>

        <div class="form-group">
          <label for="files">Archivos PDF (máx. 6MB total)</label>
          <div class="file-input-wrapper">
            <input
              id="files"
              type="file"
              multiple
              accept=".pdf"
              @change="handleFileChange"
              class="file-input"
            />
            <div class="file-info">
              <p v-if="files?.length">
                {{ files.length }} archivo(s) - {{ formatFileSize(totalSize) }}
              </p>
              <p v-else class="placeholder">Selecciona uno o más PDFs</p>
            </div>
          </div>
        </div>

        <div v-if="error" class="error-message">⚠️ {{ error }}</div>

        <button type="submit" :disabled="loading" class="btn-primary">
          {{ loading ? "Generando plan..." : "Generar Plan de Estudio" }}
        </button>
      </form>
    </div>

    <div v-else class="calendar-section">
      <button @click="reset" class="btn-secondary">← Volver</button>
      <StudyCalendar :study-plan="studyPlan" />
    </div>
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu,
    Cantarell, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
  padding: 20px;
}

.header h1 {
  font-size: 2.5em;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.header p {
  font-size: 1.1em;
  opacity: 0.95;
}

.form-section,
.calendar-section {
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  font-weight: 600;
  color: #333;
  font-size: 0.95em;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.input {
  padding: 10px 12px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 0.95em;
  transition: all 0.3s ease;
}

.input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.slots-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 10px;
}

.slot-row {
  display: grid;
  grid-template-columns: 1fr 1fr 30px 1fr 40px;
  gap: 10px;
  align-items: center;
}

.slot-row span {
  text-align: center;
  color: #666;
}

.file-input-wrapper {
  position: relative;
  border: 2px dashed #667eea;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.file-input-wrapper:hover {
  background: rgba(102, 126, 234, 0.05);
  border-color: #764ba2;
}

.file-input {
  position: absolute;
  opacity: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.file-info {
  pointer-events: none;
  color: #667eea;
  font-weight: 500;
}

.file-info .placeholder {
  color: #999;
}

.error-message {
  padding: 12px 16px;
  background: #fee;
  color: #c33;
  border-left: 4px solid #c33;
  border-radius: 4px;
  font-size: 0.95em;
}

.btn-primary,
.btn-secondary,
.btn-danger {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 0.95em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-secondary {
  background: #f0f0f0;
  color: #333;
}

.btn-secondary:hover {
  background: #e0e0e0;
}

.btn-danger {
  background: #fee;
  color: #c33;
  padding: 8px 12px;
  font-size: 0.9em;
}

.btn-danger:hover {
  background: #fdd;
}

@media (max-width: 640px) {
  .form-row {
    grid-template-columns: 1fr;
  }

  .slot-row {
    grid-template-columns: 1fr;
  }

  .header h1 {
    font-size: 1.8em;
  }
}
</style>
