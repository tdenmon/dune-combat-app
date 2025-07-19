<template>
  <img src="./assets/dune-2021-seeklogo.png">
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4 font-sans">
    <div class="bg-white p-6 rounded-xl shadow-lg w-full max-w-md">
      <h1 class="text-3xl font-bold text-center text-gray-800 mb-6">Dune Battle Calculator</h1>

      <!-- Faction Selection -->
      <div class="mb-5">
        <label for="faction-select" class="block text-gray-700 text-sm font-semibold mb-2">Select Faction:</label>
        <select
          id="faction-select"
          v-model="selectedFaction"
          class="block w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-800"
        >
          <option value="" disabled>Choose a Faction</option>
          <option v-for="faction in factions" :key="faction" :value="faction">{{ faction }}</option>
        </select>
      </div>

      <!-- Leader Selection Dropdown -->
      <div class="mb-5">
        <label for="leader-select" class="block text-gray-700 text-sm font-semibold mb-2">Select Leader:</label>
        <select
          id="leader-select"
          v-model="selectedLeaderOption"
          class="block w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-800"
          :disabled="!selectedFaction"
        >
          <option :value="null" disabled>
            {{ selectedFaction ? 'Select a Leader' : 'Select a Faction First' }}
          </option>
          <option v-if="selectedFaction" :value="null">Cheap Hero/No Leader</option>
          <option v-for="leader in availableLeaders" :key="leader.name" :value="leader">{{ leader.name }} ({{ leader.value }})</option>
        </select>
      </div>

      <!-- Leader Value Input (visible if a leader is selected or custom is chosen) -->
      <div v-if="selectedLeaderOption !== null" class="mb-5">
        <label for="leader-value" class="block text-gray-700 text-sm font-semibold mb-2">Leader Battle Value:</label>
        <input
          type="number"
          id="leader-value"
          v-model.number="leaderValue"
          min="0"
          placeholder="0"
          class="block w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-800"
        />
      </div>

      <!-- Troops Input -->
      <div class="mb-5">
        <label for="troops-committed" class="block text-gray-700 text-sm font-semibold mb-2">Troops Committed:</label>
        <input
          type="number"
          id="troops-committed"
          v-model.number="troopsCommitted"
          min="0"
          placeholder="0"
          class="block w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-800"
        />
      </div>

      <!-- Special Troops Input (Fedaykin/Sardaukar) -->
      <div v-if="selectedFaction === 'Fremen' || selectedFaction === 'Emperor'" class="mb-5">
        <label for="special-troops-committed" class="block text-gray-700 text-sm font-semibold mb-2">
          {{ selectedFaction === 'Fremen' ? 'Fedaykin' : 'Sardaukar' }} Committed:
        </label>
        <input
          type="number"
          id="special-troops-committed"
          v-model.number="specialTroopsCommitted"
          min="0"
          placeholder="0"
          class="block w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-800"
        />
      </div>

      <!-- Spice Spent Input -->
      <div v-if="selectedFaction != 'Fremen'" class="mb-6">
        <label for="spice-spent" class="block text-gray-700 text-sm font-semibold mb-2">Spice Spent:</label>
        <input
          type="number"
          id="spice-spent"
          v-model.number="spiceSpent"
          min="0"
          :max="totalTroopsCommitted"
          placeholder="0"
          class="block w-full p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-800"
        />
      </div>

      <!-- Total Power Display -->
      <div class="bg-blue-100 border border-blue-300 text-blue-800 p-4 rounded-lg text-center">
        <p class="text-lg font-semibold mb-1">Total Battle Power:</p>
        <p class="text-4xl font-extrabold">{{ totalBattlePower }}</p>
      </div>

      <!-- Reset Button -->
      <button
        @click="resetForm"
        class="mt-6 w-full bg-gray-500 hover:bg-gray-600 text-white font-bold py-3 px-4 rounded-lg focus:outline-none focus:ring-2 focus:ring-gray-400 focus:ring-opacity-75 transition duration-200"
      >
        Reset
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      selectedFaction: '',
      factions: ['Atreides', 'Harkonnen', 'Bene Gesserit', 'Spacing Guild', 'Fremen', 'Emperor'],
      factionLeaders: {
        "Atreides": [
          {"name": "Lady Jessica", "value": 5},
          {"name": "Thufir Hawat", "value": 5},
          {"name": "Gurney Halleck", "value": 4},
          {"name": "Duncan Idaho", "value": 2},
          {"name": "Dr. Wellington Yueh", "value": 1}
        ],
        "Harkonnen": [
          {"name": "Feyd-Rautha", "value": 6},
          {"name": "Beast Rabban", "value": 4},
          {"name": "Piter De Vries", "value": 3},
          {"name": "Captain Iakin Nefud", "value": 2},
          {"name": "Umman Kudu", "value": 1}
        ],
        "Bene Gesserit": [
          {"name": "Alia", "value": 5},
          {"name": "Margot Lady Fenring", "value": 5},
          {"name": "Mother Ramllo", "value": 5},
          {"name": "Princess Irulan", "value": 5},
          {"name": "Wanna Yueh", "value": 5}
        ],
        "Spacing Guild": [
          {"name": "Staban Tuek", "value": 5},
          {"name": "Master Bewt", "value": 3},
          {"name": "Esmar Tuek", "value": 3},
          {"name": "Soo-Soo Sook", "value": 2},
          {"name": "Guild Rep", "value": 1}
        ],
        "Fremen": [
          {"name": "Stilgar", "value": 7},
          {"name": "Chani", "value": 6},
          {"name": "Otheym", "value": 5},
          {"name": "Shadout Mapes", "value": 3},
          {"name": "Jamis", "value": 2}
        ],
        "Emperor": [
          {"name": "Hasimir Fenring", "value": 6},
          {"name": "Captain Aramsham", "value": 5},
          {"name": "Caid", "value": 3},
          {"name": "Burseg", "value": 3},
          {"name": "Bashar", "value": 2}
        ]
      },
      selectedLeaderOption: null,
      leaderName: '',
      leaderValue: 0,
      troopsCommitted: 0,
      specialTroopsCommitted: 0,
      spiceSpent: 0,
    };
  },
  computed: {
    totalTroopsCommitted() {
      // Calculate the sum of regular and special troops for the spice limit
      return (this.troopsCommitted > 0 ? this.troopsCommitted : 0) +
             (this.specialTroopsCommitted > 0 ? this.specialTroopsCommitted : 0);
    },
    totalBattlePower() {
      const troops = this.troopsCommitted > 0 ? this.troopsCommitted : 0;
      const specialTroops = (this.specialTroopsCommitted > 0 ? this.specialTroopsCommitted : 0);
      const leader = this.leaderValue > 0 ? this.leaderValue : 0;
      // Ensure spice spent does not exceed the total troops committed
      const effectiveSpiceSpent = Math.min(this.spiceSpent > 0 ? this.spiceSpent : 0, this.totalTroopsCommitted);
      const specialSpiceSpent = Math.min(specialTroops, effectiveSpiceSpent);
      const normalSpiceSpent = Math.max(effectiveSpiceSpent - specialSpiceSpent, 0);

      if (this.selectedFaction === 'Fremen') {
        return (troops + (specialTroops * 2)) * 2 + leader;
      }
      else {
        return troops + normalSpiceSpent + ((specialTroops + specialSpiceSpent) * 2) + leader;
      }
    },
    availableLeaders() {
      if (this.selectedFaction && this.factionLeaders[this.selectedFaction]) {
        return this.factionLeaders[this.selectedFaction];
      }
      return [];
    },
  },
  watch: {
    selectedFaction() {
      this.selectedLeaderOption = null;
      this.leaderName = '';
      this.leaderValue = 0;
      this.troopsCommitted = 0;
      this.specialTroopsCommitted = 0;
      this.spiceSpent = 0;
    },
    selectedLeaderOption(newOption) {
      if (newOption) {
        this.leaderName = newOption.name;
        this.leaderValue = newOption.value * 2;
      } else {
        this.leaderName = '';
        this.leaderValue = 0;
      }
    },
    // Watch totalTroopsCommitted to adjust spiceSpent if the limit changes
    totalTroopsCommitted(newTotalTroops) {
      if (this.spiceSpent > newTotalTroops) {
        this.spiceSpent = newTotalTroops;
      }
    },
    // Watch spiceSpent to enforce the maximum if the user manually inputs a higher value
    spiceSpent(newSpice) {
      if (newSpice > this.totalTroopsCommitted) {
        this.spiceSpent = this.totalTroopsCommitted;
      }
    }
  },
  methods: {
    resetForm() {
      this.selectedFaction = '';
      this.selectedLeaderOption = null;
      this.leaderName = '';
      this.leaderValue = 0;
      this.troopsCommitted = 0;
      this.specialTroopsCommitted = 0;
      this.spiceSpent = 0;
    },
  },
};
</script>

<style>
/* Custom font import (Inter) */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;800&display=swap');

body {
  font-family: 'Inter', sans-serif;
}
</style>
