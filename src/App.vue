<template>
  <Header></Header>
  <div class="container">
    <section class="column-1">
      <Balance :total="+total"></Balance>
      <IncomeExpenses :income="+income" :expenses="+expenses"></IncomeExpenses>
    </section>
    <section class="column-2">
      <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted"></TransactionList>
      <AddTransaction @transactionSubmitted="handleTransactionSubmitted"></AddTransaction>
    </section>
  </div>
</template>

<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import { ref, computed, onMounted } from 'vue';

import { useToast } from 'vue-toastification';

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  saveStorage();
});

/* Functions */
function saveStorage() {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

  if(savedTransactions) {
    transactions.value = savedTransactions;
  }
}

// Get total
const total = computed(() => {
  return transactions.value.reduce((sum, transaction) => {
    return sum + transaction.amount;
  }, 0);
});

// Get income
const income = computed(() => {
  return transactions.value
  .filter((transaction) => transaction.amount > 0)
  .reduce((sum, transaction) => {
    return sum + transaction.amount;
  }, 0).toFixed(2);
});

// Get expenses
const expenses = computed(() => {
  return transactions.value
  .filter((transaction) => transaction.amount < 0)
  .reduce((sum, transaction) => {
    return sum + transaction.amount;
  }, 0).toFixed(2);
});

// Add transaction
const handleTransactionSubmitted = (transactionData) => {
  transactions.value.push({
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount,
  });

  saveTransactionsToLocalStorage();

  toast.success('Transaction added successfully');
}

// Generate unique ID
const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000);
}

// Delete transaction
const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter((transaction) => transaction.id !== id);
  toast.success('Transaction deleted successfully');

  saveTransactionsToLocalStorage()
}

// Save to localstorage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
}
</script>

<style scoped></style>
