<template>
  <div id="app">
    <div class="header">
      <h1>Pemilihan Pemimpin Kelas</h1>
    </div>
    <div class="input-group">
      <input
        v-model="newCandidate"
        placeholder="Tambahkan calon pemimpin ....."
        @keypress.enter="addCandidateOnEnter"
      />
      <button @click="addCandidate">
        <font-awesome-icon :icon="['fas', 'plus']" class="icon" />
        Tambah
      </button>
    </div>
    <div class="candidates">
      <div v-if="!userHasVoted">
        <div v-for="candidate in candidates" :key="candidate.id" class="candidate">
          <button @click="vote(candidate.id)">
            <font-awesome-icon icon="user" class="icon" />
            <span class="candidate-name">{{ candidate.name }}</span>
          </button>
        </div>
      </div>
      <div v-else class="thank-you">
        <p>Terima kasih telah memilih!</p>
      </div>
    </div>
    <div class="vote-count">
      <h2>Hasil Voting</h2>
      <div v-for="candidate in candidates" :key="candidate.id" class="result">
        <span>{{ candidate.name }}:</span> <span>{{ candidate.votes }}</span>
      </div>
    </div>
    <!-- <div class="actions">
      <button @click="resetVotes">Reset Voting</button>
    </div> -->
    <div class="copyright">
      <p>&copy; 2024 <a href="https://www.github.com/AlpianPPLG">Alpian</a> All rights reserved.</p>
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2'

export default {
  data() {
    return {
      newCandidate: '',
      candidates: [],
      userHasVoted: false
    }
  },
  methods: {
    addCandidate() {
      if (this.newCandidate.trim() === '') {
        Swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Nama calon tidak boleh kosong.'
        })
        return
      }
      if (this.candidateExists(this.newCandidate.trim())) {
        Swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Nama calon sudah ada.'
        })
        return
      }
      this.candidates.push({ id: Date.now(), name: this.newCandidate.trim(), votes: 0 })
      this.newCandidate = ''
    },
    addCandidateOnEnter(event) {
      if (event.key === 'Enter') {
        this.addCandidate()
      }
    },
    candidateExists(name) {
      return this.candidates.some(
        (candidate) => candidate.name.toLowerCase() === name.toLowerCase()
      )
    },
    vote(candidateId) {
      if (!this.userHasVoted) {
        const candidate = this.candidates.find((c) => c.id === candidateId)
        if (candidate) {
          candidate.votes++
          this.userHasVoted = true
        }
      }
    }
    // resetVotes() {
    //   this.candidates.forEach((candidate) => (candidate.votes = 0))
    //   this.userHasVoted = false
    // }
  }
}
</script>

<style scoped>
#app {
  font-family: 'Roboto', sans-serif;
  background-color: #f5f5f5;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  box-sizing: border-box;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 90%;
  max-width: 600px;
  margin: 50px auto;
  text-align: center;
  background: #ffffff;
}

.header {
  background-color: #6200ea;
  color: white;
  padding: 20px;
  border-radius: 8px 8px 0 0;
  width: 100%;
}

h1 {
  font-size: 24px;
  margin: 0;
}

.input-group {
  display: flex;
  justify-content: center;
  margin: 20px 0;
  width: 100%;
}

input {
  padding: 10px;
  font-size: 16px;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 70%;
}

button {
  background-color: #6200ea;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition:
    background-color 0.3s,
    transform 0.1s;
}

button .icon {
  margin-right: 8px; /* Adjust the margin between icon and text */
}

button:hover {
  background-color: #3700b3;
}

button:active {
  background-color: #03dac5;
  transform: scale(0.98);
}

.candidates {
  width: 100%;
}

.candidate {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #fafafa;
  transition: transform 0.2s;
}

.candidate:hover {
  transform: translateY(-2px);
}

.vote-count {
  font-size: 18px;
  margin-top: 20px;
  width: 100%;
}

.vote-count h2 {
  margin-bottom: 10px;
}

.result {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #e0e0e0;
  margin: 5px 0;
}

.actions {
  margin-top: 20px;
}

.thank-you {
  color: #6200ea;
  font-size: 20px;
  margin: 20px 0;
}

.candidate-name {
  margin-left: 4px; /* Adjust the margin between icon and text */
}

/* Styling for copyright */
.copyright {
  margin-top: 20px;
  font-size: 15px;
  color: black;
  text-align: center;
}
</style>
