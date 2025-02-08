<template>
  <div class="d-flex align-center justify-center" style="height: 100vh">
    <v-container class="mx-auto" max-width="400">
      <v-row align="center" justify="center">
        <v-col>
          <v-card >
            <v-card-title>Time Since Last</v-card-title>
            <v-card-subtitle>{{
              selectedDate && selectedTime
                ? `${timeSinceEntered.days} days, ${timeSinceEntered.hours} hours, ${timeSinceEntered.minutes} minutes, ${timeSinceEntered.seconds} seconds`
                : "Please add data using show fields"
            }}</v-card-subtitle>
            <v-card-subtitle class="mt-3" v-if="selectedDate && selectedTime"> 
            Streak : {{streak}}
            </v-card-subtitle>
            <v-card-text>
              <div v-if="showFields">
                <v-text-field
                  v-model="selectedDate"
                  label="Enter Date"
                  type="date"
                >
                </v-text-field>
                <v-text-field
                  v-model="selectedTime"
                  label="Enter time"
                  type="time"
                >
                </v-text-field>

              </div>
            </v-card-text>

            <v-card-actions>
              <v-btn color="secondary" variant="text" @click="toggleFields"
                >Update Data</v-btn
              >
              <v-btn
                color="secondary"
                variant="text"
                @click="exportDateTimeLocal"
                >Export</v-btn
              >

              <v-btn
                color="secondary"
                variant="text"
                @click="failedAttempts"
                >Oops!</v-btn
              >
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>

 
  <div class="mt-5">
  <v-table >
    <thead>
      <tr>
        <th class="text-left">
          Streak
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="item in failedDatesDescending"
        :key="item.name"
      >
        <td>{{ item.streak }}</td>
      </tr>
    </tbody>
  </v-table>
</div>
</v-container>

</div>

</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";


// Added reactive variables for date and time
const selectedDate = ref(null);
const selectedTime = ref(null);
const currentTime = ref(new Date());
const showFields = ref(false);

const failedDates = ref([])

const failedAttempts = () => {

  failedDates.value.push({streak: streak.value.slice(0, -2) + " 🏆"})
  localStorage.setItem("failedDates", JSON.stringify(failedDates.value))
  resetDateTime()

}

const saveDateTimeToLocalStorage = () => {
  localStorage.setItem("selectedDate", selectedDate.value);
  localStorage.setItem("selectedTime", selectedTime.value);
};

watch(selectedDate, saveDateTimeToLocalStorage);
watch(selectedTime, saveDateTimeToLocalStorage);

const toggleFields = () => {
  showFields.value = !showFields.value;
};

// Function to export date and time to a text file
const exportDateTimeLocal = () => {
  const dateTime = `Date: ${selectedDate.value}\nTime: ${selectedTime.value}\nFailed Dates: ${JSON.stringify(failedDates.value, null, 2)}`;
  const blob = new Blob([dateTime], { type: "text/plain" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "dateTime.txt";
  link.click();
}; // Added reactive variables for date and time

const timeSinceEntered = computed(() => {
  if (!selectedDate.value || !selectedTime.value) return { days: 0, hours: 0, minutes: 0, seconds: 0 };

  const enteredDateTime = new Date(`${selectedDate.value}T${selectedTime.value}`);
  const diffTime = Math.abs(currentTime.value - enteredDateTime);

  const days = Math.floor(diffTime / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diffTime % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((diffTime % (1000 * 60 * 60)) / (1000 * 60));
  const seconds = Math.floor((diffTime % (1000 * 60)) / 1000);

  return { days, hours, minutes, seconds };
});

const streak = computed(() => {
    return `${timeSinceEntered.value.days} days 🔥`
})


const resetDateTime = () => {
  selectedDate.value = new Date().toISOString().substr(0, 10);
  selectedTime.value = new Date().toTimeString().substr(0, 5);
}

const failedDatesDescending =computed( () => {
  return failedDates.value.slice(-10).reverse()
})

let intervalId;
onMounted(() => {
  intervalId = setInterval(() => {
    currentTime.value = new Date();
  }, 1000);

  // Load date and time from localStorage
  selectedDate.value = localStorage.getItem("selectedDate") || null;
  selectedTime.value = localStorage.getItem("selectedTime") || null;
  failedDates.value = JSON.parse(localStorage.getItem("failedDates")) || []
});

onUnmounted(() => {
  clearInterval(intervalId);
});
</script>