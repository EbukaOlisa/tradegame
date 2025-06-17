<template>
  <div class="details-pan">
    <h2>Investment Details</h2>
    <div v-if="!displayedInvestments || displayedInvestments.length === 0" class="no-investments">
      Loading Investments.
    </div>
    <div v-else>
      <ul>
        <li
          v-for="investment in displayedInvestments"
          :key="investment._id"
          class="investment-item"
        >
          <span class="category">
          {{ investment.category }}
          <span v-if="investment.isDemo" class="demo-label"> (Demo)</span>
          </span>

          <span class="outcome">{{ investment.outcome || 'Pending' }}</span>
          <span class="status">{{ investment.status }}</span>
          <details>
            <summary>Details</summary>
            <p><strong>Choice:</strong> {{ investment.choice }}</p>
            <p><strong>Amount:</strong> ₦{{ investment.amount }}</p>
            <p><strong>Odds:</strong> {{ investment.odds }}</p>
            <p><strong>Timeframe:</strong> {{ investment.timeframe }}</p>
          </details>
        </li>
      </ul>
      <button v-if="hasMore" @click="loadMoreInvestments">Load More</button>
    </div>
  </div>
</template>
<script>
import { mapGetters } from "vuex";
import axios from "axios";

export default {
  name: "InvestmentDetails",
  data() {
    return {
      realInvestments: [],
      demoInvestments: [],
      displayedInvestments: [],
      itemsPerPage: 70,
      currentPage: 0,
      hasMore: true,
    };
  },
  computed: {
    ...mapGetters(["userId", "token"]),
  },
  methods: {
    async fetchInvestments() {
      if (!this.userId) return;

      try {
        const response = await axios.get(
          `${import.meta.env.VITE_APP_BASE_URL}/api/investments/user/${this.userId}`,
          {
            headers: {
              Authorization: `Bearer ${this.token}`,
            },
          }
        );

        this.realInvestments = (response.data || []).map(item => ({
          ...item,
          isDemo: false,
        }));
      } catch (error) {
        console.error("Error fetching real investments:", error);
      }

      try {
        const demoRes = await axios.get(
          `${import.meta.env.VITE_APP_BASE_URL}/api/investments/demouser/${this.userId}`,
          {
            headers: {
              Authorization: `Bearer ${this.token}`,
            },
          }
        );

        this.demoInvestments = (demoRes.data || []).map(item => ({
          ...item,
          isDemo: true,
        }));
      } catch (error) {
        console.error("Error fetching demo investments:", error);
      }

      const all = [...this.realInvestments, ...this.demoInvestments].sort(
        (a, b) => new Date(b.createdAt) - new Date(a.createdAt)
      );

      this.investments = all;
      this.updateDisplayedInvestments();
    },

    updateDisplayedInvestments() {
      const start = this.currentPage * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      const newBatch = this.investments.slice(start, end);

      this.displayedInvestments = [...this.displayedInvestments, ...newBatch];

      if (end >= this.investments.length) {
        this.hasMore = false;
      }
    },

    loadMoreInvestments() {
      this.currentPage++;
      this.updateDisplayedInvestments();
    },
  },

  created() {
    this.fetchInvestments();
  },
};
</script>



<style scoped>
h2 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 20px;
}

.no-investments {
  color: red;
  text-align: center;
  font-size: 18px;
  margin-top: 20px;
}

ul {
  list-style-type: none;
  padding: 0;
}

.investment-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5.5px;
  border: 1px solid #ccc;
  border-radius: 8px;
  background: linear-gradient(135deg, #003366, green);
  color: white;
  margin-bottom: 10px;
}

.investment-item:hover {
  background: linear-gradient(135deg,rgb(55, 104, 121), #2e86de);
}

.category {
  flex: 1;
  text-align: left;
  font-weight: bold;
  color: yellow;
}

.outcome {
  flex: 1;
  text-align: center;
  font-weight: bold;
  color: yellow;
}

.status {
  flex: 1;
  text-align: right;
  font-weight: bold;
  color: yellow;
}

details summary {
  margin-top: 10px;
  cursor: pointer;
  font-weight: bold;
  color: #fff;
}

details summary:hover {
  text-decoration: underline;
}

details p {
  color: #fff;
  margin: 5px 0;
}

button {
  margin: 20px auto;
  display: block;
  background: #2e86de;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.details-pan{
  text-align: center;
  margin: 20px auto;
 
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: lightyellow;
}

button:hover {
  background: #48c6ef;
}
.demo-label {
  color: red;
  font-weight: bold;
  font-size: 0.9rem;
}

</style>
