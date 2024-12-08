<template>
  <div id="app" :class="{ dark: isDarkMode }">
    <div class="header">
      <h1>{{ currentLanguage.title }}</h1>
      <button @click="toggleDarkMode">
        {{ isDarkMode ? currentLanguage.lightMode : currentLanguage.darkMode }}
      </button>
      <select v-model="selectedLanguage" @change="changeLanguage">
        <option v-for="(lang, index) in languages" :key="index" :value="lang.code">
          {{ lang.name }}
        </option>
      </select>
      <button @click="exportToPDF">{{ currentLanguage.exportPDF }}</button>
    </div>
    <div class="input-group">
      <input
        v-model="newCandidate"
        :placeholder="currentLanguage.addCandidatePlaceholder"
        @keypress.enter="addCandidateOnEnter"
      />
      <button @click="addCandidate">
        <font-awesome-icon :icon="['fas', 'plus']" class="icon" />
        {{ currentLanguage.add }}
      </button>
    </div>
    <div class="candidates">
      <div v-if="!userHasVoted">
        <div v-for="candidate in candidates" :key="candidate.id" class="candidate">
          <button @click="vote(candidate.id)">
            <font-awesome-icon icon="user" class="icon" />
            <span class="candidate-name">{{ candidate.name }}</span>
          </button>
          <button @click="support(candidate.id)">
            <font-awesome-icon icon="thumbs-up" class="icon" />
            {{ currentLanguage.support }}
          </button>
        </div>
      </div>
      <div v-else class="thank-you">
        <p>{{ currentLanguage.thankYou }}</p>
      </div>
    </div>
    <div class="vote-count">
      <h2>{{ currentLanguage.voteResults }}</h2>
      <div v-for="candidate in candidates" :key="candidate.id" class="result">
        <span>{{ candidate.name }}:</span> <span>{{ candidate.votes }}</span>
      </div>
      <h2>{{ currentLanguage.statistics }}</h2>
      <div class="statistics">
        <div v-for="candidate in candidates" :key="candidate.id">
          <span
            >{{ candidate.name }} - {{ ((candidate.votes / totalVotes) * 100).toFixed(2) }}%</span
          >
        </div>
      </div>
    </div>
    <div class="education">
      <h2>{{ currentLanguage.education }}</h2>
      <div v-for="candidate in candidates" :key="candidate.id" class="candidate-info">
        <h3>{{ candidate.name }}</h3>
        <p>{{ candidate.info }}</p>
      </div>
    </div>
    <div class="comments">
      <h2>{{ currentLanguage.comments }}</h2>
      <input
        v-model="newComment"
        :placeholder="currentLanguage.commentPlaceholder"
        @keypress.enter="addComment"
      />
      <div v-for="comment in comments" :key="comment.id" class="comment">
        <p>{{ comment.text }}</p>
      </div>
    </div>
    <div class="copyright">
      <p>&copy; 2024 <a href="https://www.github.com/AlpianPPLG">Alpian</a> All rights reserved.</p>
    </div>
    <div v-if="reminderMessage" class="reminder">
      <p>{{ reminderMessage }}</p>
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
import jsPDF from 'jspdf'
import autoTable from 'jspdf-autotable'
import { ref, onMounted } from 'vue'

export default {
  setup() {
    const newCandidate = ref('')
    const candidates = ref([])
    const userHasVoted = ref(false)
    const newComment = ref('')
    const comments = ref([])
    const isDarkMode = ref(false)
    const selectedLanguage = ref('en')
    const languages = ref([
      { code: 'en', name: 'English' },
      { code: 'id', name: 'Bahasa Indonesia' }
    ])
    const translations = {
      en: {
        title: 'Class Leader Election',
        lightMode: 'Light Mode',
        darkMode: 'Dark Mode',
        addCandidatePlaceholder: 'Add a candidate...',
        add: 'Add',
        support: 'Support',
        thankYou: 'Thank you for voting!',
        voteResults: 'Voting Results',
        statistics: 'Statistics',
        education: 'Candidate Information',
        comments: 'Comments',
        commentPlaceholder: 'Write a comment...',
        exportPDF: 'Export to PDF',
        reminderMessage: 'Don’t forget to vote before the deadline!'
      },
      id: {
        title: 'Pemilihan Pemimpin Kelas',
        lightMode: 'Tema Terang',
        darkMode: 'Tema Gelap',
        addCandidatePlaceholder: 'Tambahkan calon...',
        add: 'Tambah',
        support: 'Dukung',
        thankYou: 'Terima kasih telah memilih!',
        voteResults: 'Hasil Voting',
        statistics: 'Statistik',
        education: 'Informasi Calon',
        comments: 'Komentar',
        commentPlaceholder: 'Tulis komentar...',
        exportPDF: 'Ekspor ke PDF',
        reminderMessage: 'Jangan lupa untuk memilih sebelum batas waktu!'
      }
    }

    const reminderMessage = ref('')

    const totalVotes = () => {
      return candidates.value.reduce((total, candidate) => total + candidate.votes, 0)
    }

    const currentLanguage = () => {
      return translations[selectedLanguage.value]
    }

    const toggleDarkMode = () => {
      isDarkMode.value = !isDarkMode.value
    }

    const changeLanguage = () => {
      // This method is triggered when the user selects a different language.
    }

    const addCandidate = () => {
      if (newCandidate.value.trim() === '') {
        Swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: currentLanguage().addCandidatePlaceholder
        })
        return
      }
      if (candidateExists(newCandidate.value.trim())) {
        Swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Candidate already exists.'
        })
        return
      }
      candidates.value.push({
        id: Date.now(),
        name: newCandidate.value.trim(),
        votes: 0,
        info: 'Informasi tentang calon...' // Placeholder for candidate info
      })
      newCandidate.value = ''
    }

    const addCandidateOnEnter = (event) => {
      if (event.key === 'Enter') {
        addCandidate()
      }
    }

    const candidateExists = (name) => {
      return candidates.value.some(
        (candidate) => candidate.name.toLowerCase() === name.toLowerCase()
      )
    }

    const vote = (candidateId) => {
      if (!userHasVoted.value) {
        const candidate = candidates.value.find((c) => c.id === candidateId)
        if (candidate) {
          candidate.votes++
          userHasVoted.value = true
        }
      }
    }

    const support = (candidateId) => {
      Swal.fire({
        icon: 'success',
        title: 'Support Received',
        text: `You supported ${candidates.value.find((c) => c.id === candidateId).name}`
      })
    }

    const addComment = () => {
      if (newComment.value.trim() !== '') {
        comments.value.push({ id: Date.now(), text: newComment.value.trim() })
        newComment.value = ''
      }
    }

    const exportToPDF = () => {
      const doc = new jsPDF()
      doc.text(currentLanguage().title, 14, 16)
      const tableData = candidates.value.map((candidate) => [candidate.name, candidate.votes])
      autoTable(doc, {
        head: [['Candidate', 'Votes']],
        body: tableData,
        startY: 30
      })
      doc.save('Voting_Results.pdf')
    }

    // Reminder functionality
    const setReminder = () => {
      reminderMessage.value = currentLanguage().reminderMessage
      setTimeout(() => {
        reminderMessage.value = ''
      }, 10000) // Show reminder for 10 seconds
    }

    onMounted(() => {
      setReminder()
    })

    return {
      newCandidate,
      candidates,
      userHasVoted,
      newComment,
      comments,
      isDarkMode,
      selectedLanguage,
      languages,
      currentLanguage,
      totalVotes,
      toggleDarkMode,
      changeLanguage,
      addCandidate,
      addCandidateOnEnter,
      candidateExists,
      vote,
      support,
      addComment,
      exportToPDF,
      reminderMessage
    }
  }
}
</script>

<style scoped>
#app {
  font-family: 'Roboto', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  box-sizing: border-box;
  border-radius: 8px;
  width: 90%;
  max-width: 600px;
  margin: 50px auto;
  text-align: center;
}

.dark {
  background-color: #121212;
  color: white;
}

.header {
  background-color: #6200ea;
  color: white;
  padding: 20px;
  border-radius: 8px 8px 0 0;
  width: 100%;
}

button {
  margin-left: 10px;
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

.candidates {
  width: 100%;
}

.candidate {
  display: flex;
  justify-content: space-between;
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

.statistics {
  margin-top: 10px;
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

.education {
  margin-top: 20px;
  width: 100%;
}

.candidate-info {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin: 5px 0;
}

.comments {
  margin-top: 20px;
  width: 100%;
}

.comment {
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin: 5px 0;
}

.copyright {
  margin-top: 20px;
  font-size: 15px;
  color: black;
  text-align: center;
}

.reminder {
  margin-top: 20px;
  font-weight: bold;
  color: red;
}
</style>
