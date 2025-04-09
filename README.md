<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=0.6" />
  <title>Fit Zone - Gym Members</title>
  <style>
    * {
      box-sizing: border-box;
    }
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f4f4f4;
    }
    header {
      background-color: #000;
      color: white;
      padding: 1rem;
      text-align: center;
    }
    nav {
      display: flex;
      justify-content: space-around;
      background-color: #000;
      padding: 0.5rem;
      flex-wrap: wrap;
    }
    nav button {
      background-color: #444;
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      margin: 0.2rem;
      cursor: pointer;
      border-radius: 5px;
      font-size: 0.9rem;
    }
    .container {
      padding: 1rem;
    }
    .member-card {
      background-color: white;
      border-left: 6px solid #444;
      padding: 1rem;
      margin: 1rem 0;
      border-radius: 5px;
      box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    }
    .member-card.pending {
      background-color: #ffe6e6;
      border-left: 6px solid red;
    }
    .status {
      font-weight: bold;
    }
    .status.active {
      color: green;
    }
    .status.pending {
      color: red;
    }
    .actions button {
      margin: 0.2rem;
      padding: 0.3rem 0.6rem;
      font-size: 0.8rem;
    }
    input, select {
      margin: 0.2rem;
      padding: 0.5rem;
      font-size: 1rem;
      width: 100%;
      max-width: 300px;
    }
    .dashboard {
      margin: 1rem 0;
      padding: 1rem;
      background-color: #e6e6e6;
      border-radius: 5px;
    }
    @media (max-width: 600px) {
      nav button {
        font-size: 0.8rem;
        padding: 0.4rem;
      }
    }
  </style>
</head>
<body>
  <header>
    <h1>Fit Zone</h1>
  </header>
  <nav>
    <button onclick="showForm()">Add Member</button>
    <button onclick="showDashboard()">Dashboard</button>
    <button onclick="showIncome()">Total Income</button>
  </nav>
  <div class="container" id="main-content"></div>

  <script>
    let members = JSON.parse(localStorage.getItem('members')) || [];

    function saveMembers() {
      localStorage.setItem('members', JSON.stringify(members));
    }

    function showForm() {
      document.getElementById('main-content').innerHTML = `
        <h2>Add New Member</h2>
        <input type="text" id="name" placeholder="Member Name" />
        <input type="number" id="mobile" placeholder="Mobile Number" />
        <input type="number" id="amount" placeholder="Amount Paid" />
        <input type="date" id="date" />
        <select id="duration">
          <option value="1">1 Month</option>
          <option value="2">2 Months</option>
          <option value="3">3 Months</option>
          <option value="4">4 Months</option>
          <option value="5">5 Months</option>
          <option value="6">6 Months</option>
          <option value="12">1 Year</option>
        </select>
        <button onclick="addMember()">Add Member</button>
        <hr />
        <input type="text" id="search" placeholder="Search Pending Members" oninput="renderMembers()" />
        <div id="members-list"></div>
      `;
      renderMembers();
    }

    function addMember() {
      const name = document.getElementById('name').value;
      const mobile = document.getElementById('mobile').value;
      const amount = document.getElementById('amount').value;
      const date = document.getElementById('date').value;
      const duration = parseInt(document.getElementById('duration').value);

      if (!name || !mobile || !amount || !date) return alert('Please fill all fields');

      const expiry = new Date(date);
      expiry.setMonth(expiry.getMonth() + duration);

      members.push({ name, mobile, amount, date, expiry: expiry.toISOString(), status: 'active' });
      saveMembers();
      showForm();
    }

    function renderMembers() {
      const list = document.getElementById('members-list');
      if (!list) return;
      const search = document.getElementById('search').value.toLowerCase();
      list.innerHTML = '';

      const today = new Date();
      members.forEach((m, i) => {
        const isPending = new Date(m.expiry) < today;
        if (search && (!isPending || !m.name.toLowerCase().includes(search))) return;

        const cardClass = isPending ? 'member-card pending' : 'member-card';
        const statusClass = isPending ? 'status pending' : 'status active';

        list.innerHTML += `
          <div class="${cardClass}">
            <p><strong>Name:</strong> ${m.name}</p>
            <p><strong>Mobile:</strong> ${m.mobile}</p>
            <p><strong>Amount:</strong> ₹${m.amount}</p>
            <p><strong>Date:</strong> ${m.date}</p>
            <p><strong>Status:</strong> <span class="${statusClass}">${isPending ? 'Pending' : 'Active'}</span></p>
            <div class="actions">
              <button onclick="deleteMember(${i})">Delete</button>
              ${isPending ? `<button onclick="markPaid(${i})">Mark as Paid</button>` : ''}
            </div>
          </div>
        `;
      });
    }

    function deleteMember(index) {
      members.splice(index, 1);
      saveMembers();
      showForm();
    }

    function markPaid(index) {
      const today = new Date().toISOString().split('T')[0];
      members[index].date = today;
      members[index].expiry = new Date(new Date().setMonth(new Date().getMonth() + 1));
      members[index].status = 'active';
      saveMembers();
      showForm();
    }

    function showDashboard() {
      const today = new Date();
      const active = members.filter(m => new Date(m.expiry) >= today);
      const pending = members.filter(m => new Date(m.expiry) < today);
      const outstandingAmount = pending.reduce((total, m) => total + Number(m.amount), 0);

      let html = `
        <div class="dashboard">
          <h2>Dashboard</h2>
          <p><strong>Total Pending Members:</strong> ${pending.length}</p>
          <p><strong>Total Outstanding Amount:</strong> ₹${outstandingAmount}</p>
          <h3>Active Members:</h3>
          <ul>${active.map(m => `<li>${m.name}</li>`).join('')}</ul>
          <h3>Pending Members:</h3>
          <ol>${pending.map(m => `<li>${m.name}</li>`).join('')}</ol>
        </div>
      `;
      document.getElementById('main-content').innerHTML = html;
    }

    function showIncome() {
      const totalIncome = members.reduce((sum, m) => sum + Number(m.amount), 0);
      document.getElementById('main-content').innerHTML = `
        <div class="dashboard">
          <h2>Total Income</h2>
          <p><strong>Total Members:</strong> ${members.length}</p>
          <p><strong>Total Amount Collected:</strong> ₹${totalIncome}</p>
        </div>
      `;
    }

    showForm();
  </script>
</body>
</html>
