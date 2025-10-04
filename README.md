import React, { useState, useEffect } from 'react';
import { LineChart, Line, PieChart, Pie, Cell, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import { DollarSign, TrendingUp, PiggyBank, Sparkles } from 'lucide-react';

// Mock database - In production, replace with actual backend API calls
const mockDB = {
  users: [],
  income: [],
  expenses: [],
  savingsGoals: []
};

const CATEGORIES = ['Food', 'Rent', 'Subscriptions', 'Fun', 'Other'];
const COLORS = ['#2ecc71', '#3498db', '#9b59b6', '#f39c12', '#e74c3c'];

export default function SmartBudgetApp() {
  const [currentUser, setCurrentUser] = useState(null);
  const [page, setPage] = useState('login');
  const [income, setIncome] = useState([]);
  const [expenses, setExpenses] = useState([]);
  const [savingsGoals, setSavingsGoals] = useState([]);
  const [aiInsights, setAiInsights] = useState('');
  const [aiLoading, setAiLoading] = useState(false);

  // Auth states
  const [authMode, setAuthMode] = useState('login');
  const [authForm, setAuthForm] = useState({ name: '', email: '', password: '' });

  // Form states
  const [incomeForm, setIncomeForm] = useState({ source: '', amount: '', date: '' });
  const [expenseForm, setExpenseForm] = useState({ category: 'Food', amount: '', date: '' });
  const [goalForm, setGoalForm] = useState({ goal_name: '', target_amount: '' });

  useEffect(() => {
    if (currentUser) {
      loadUserData();
    }
  }, [currentUser]);

  const loadUserData = () => {
    setIncome(mockDB.income.filter(i => i.user_id === currentUser.id));
    setExpenses(mockDB.expenses.filter(e => e.user_id === currentUser.id));
    setSavingsGoals(mockDB.savingsGoals.filter(g => g.user_id === currentUser.id));
  };

  const handleAuth = (e) => {
    e.preventDefault();
    if (authMode === 'signup') {
      const newUser = {
        id: Date.now().toString(),
        name: authForm.name,
        email: authForm.email,
        password: authForm.password
      };
      mockDB.users.push(newUser);
      setCurrentUser(newUser);
      setPage('dashboard');
    } else {
      const user = mockDB.users.find(u => u.email === authForm.email && u.password === authForm.password);
      if (user) {
        setCurrentUser(user);
        setPage('dashboard');
      } else {
        alert('Invalid credentials');
      }
    }
  };

  const addIncome = (e) => {
    e.preventDefault();
    const newIncome = {
      id: Date.now().toString(),
      user_id: currentUser.id,
      ...incomeForm,
      amount: parseFloat(incomeForm.amount)
    };
    mockDB.income.push(newIncome);
    setIncome([...income, newIncome]);
    setIncomeForm({ source: '', amount: '', date: '' });
  };

  const addExpense = (e) => {
    e.preventDefault();
    const newExpense = {
      id: Date.now().toString(),
      user_id: currentUser.id,
      ...expenseForm,
      amount: parseFloat(expenseForm.amount)
    };
    mockDB.expenses.push(newExpense);
    setExpenses([...expenses, newExpense]);
    setExpenseForm({ category: 'Food', amount: '', date: '' });
  };

  const addGoal = (e) => {
    e.preventDefault();
    const newGoal = {
      id: Date.now().toString(),
      user_id: currentUser.id,
      ...goalForm,
      target_amount: parseFloat(goalForm.target_amount)
    };
    mockDB.savingsGoals.push(newGoal);
    setSavingsGoals([...savingsGoals, newGoal]);
    setGoalForm({ goal_name: '', target_amount: '' });
  };

  const getCurrentMonthData = () => {
    const now = new Date();
    const currentMonth = now.getMonth();
    const currentYear = now.getFullYear();
    
    const monthIncome = income.filter(i => {
      const d = new Date(i.date);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    });
    
    const monthExpenses = expenses.filter(e => {
      const d = new Date(e.date);
      return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
    });
    
    const totalIncome = monthIncome.reduce((sum, i) => sum + i.amount, 0);
    const totalExpenses = monthExpenses.reduce((sum, e) => sum + e.amount, 0);
    const savingsRate = totalIncome > 0 ? ((totalIncome - totalExpenses) / totalIncome * 100) : 0;
    
    return { totalIncome, totalExpenses, savingsRate, monthIncome, monthExpenses };
  };

  const getExpensesByCategory = () => {
    const { monthExpenses } = getCurrentMonthData();
    const categoryTotals = {};
    CATEGORIES.forEach(cat => categoryTotals[cat] = 0);
    monthExpenses.forEach(e => categoryTotals[e.category] += e.amount);
    return Object.entries(categoryTotals).map(([name, value]) => ({ name, value }));
  };

  const getIncomeVsExpenses = () => {
    const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'];
    return months.map(month => {
      const inc = Math.random() * 5000 + 3000;
      const exp = Math.random() * 4000 + 2000;
      return { month, income: inc, expenses: exp };
    });
  };

  const getSavingsProgress = () => {
    const { totalIncome, totalExpenses } = getCurrentMonthData();
    const saved = totalIncome - totalExpenses;
    if (savingsGoals.length === 0) return { goal: null, progress: 0, saved: 0 };
    const goal = savingsGoals[0];
    const progress = (saved / goal.target_amount) * 100;
    return { goal, progress: Math.min(progress, 100), saved };
  };

  const getAIInsights = async () => {
    setAiLoading(true);
    const { totalIncome, totalExpenses, monthExpenses } = getCurrentMonthData();
    const { goal, saved } = getSavingsProgress();
    
    // Simulate AI response (replace with actual OpenAI API call)
    setTimeout(() => {
      const categoryBreakdown = getExpensesByCategory().sort((a, b) => b.value - a.value);
      const topCategory = categoryBreakdown[0];
      
      const insights = `**Your Financial Analysis:**

📊 Your biggest spending category is **${topCategory.name}** at $${topCategory.value.toFixed(2)}. Consider reducing this by 15% to save an extra $${(topCategory.value * 0.15).toFixed(2)}/month.

💰 With your current savings rate of ${((totalIncome - totalExpenses) / totalIncome * 100).toFixed(1)}%, ${goal ? `you'll reach your "${goal.goal_name}" goal in approximately ${Math.ceil(goal.target_amount / saved)} months` : 'consider setting a savings goal to stay motivated'}.

🎯 Quick win: Review your ${categoryBreakdown[1].name} expenses. Even small cuts here can compound into significant savings over time!`;
      
      setAiInsights(insights);
      setAiLoading(false);
    }, 2000);
  };

  if (!currentUser) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-blue-50 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-md">
          <div className="text-center mb-8">
            <h1 className="text-4xl font-bold text-gray-800 mb-2">SmartBudget</h1>
            <p className="text-gray-600">Track. Save. Grow.</p>
          </div>
          
          <form onSubmit={handleAuth} className="space-y-4">
            {authMode === 'signup' && (
              <input
                type="text"
                placeholder="Full Name"
                className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-transparent"
                value={authForm.name}
                onChange={(e) => setAuthForm({...authForm, name: e.target.value})}
                required
              />
            )}
            <input
              type="email"
              placeholder="Email"
              className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-transparent"
              value={authForm.email}
              onChange={(e) => setAuthForm({...authForm, email: e.target.value})}
              required
            />
            <input
              type="password"
              placeholder="Password"
              className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-transparent"
              value={authForm.password}
              onChange={(e) => setAuthForm({...authForm, password: e.target.value})}
              required
            />
            <button
              type="submit"
              className="w-full bg-green-500 text-white py-3 rounded-lg font-semibold hover:bg-green-600 transition"
            >
              {authMode === 'login' ? 'Login' : 'Sign Up'}
            </button>
          </form>
          
          <p className="text-center mt-4 text-gray-600">
            {authMode === 'login' ? "Don't have an account? " : "Already have an account? "}
            <button
              onClick={() => setAuthMode(authMode === 'login' ? 'signup' : 'login')}
              className="text-green-500 font-semibold hover:underline"
            >
              {authMode === 'login' ? 'Sign Up' : 'Login'}
            </button>
          </p>
        </div>
      </div>
    );
  }

  const { totalIncome, totalExpenses, savingsRate } = getCurrentMonthData();
  const { goal, progress, saved } = getSavingsProgress();

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Navigation */}
      <nav className="bg-white shadow-sm border-b">
        <div className="max-w-7xl mx-auto px-4 py-4 flex items-center justify-between">
          <h1 className="text-2xl font-bold text-gray-800">SmartBudget</h1>
          <div className="flex gap-4">
            <button onClick={() => setPage('dashboard')} className={`px-4 py-2 rounded-lg ${page === 'dashboard' ? 'bg-green-500 text-white' : 'text-gray-600 hover:bg-gray-100'}`}>Dashboard</button>
            <button onClick={() => setPage('income')} className={`px-4 py-2 rounded-lg ${page === 'income' ? 'bg-green-500 text-white' : 'text-gray-600 hover:bg-gray-100'}`}>Income</button>
            <button onClick={() => setPage('expenses')} className={`px-4 py-2 rounded-lg ${page === 'expenses' ? 'bg-green-500 text-white' : 'text-gray-600 hover:bg-gray-100'}`}>Expenses</button>
            <button onClick={() => setPage('goals')} className={`px-4 py-2 rounded-lg ${page === 'goals' ? 'bg-green-500 text-white' : 'text-gray-600 hover:bg-gray-100'}`}>Goals</button>
            <button onClick={() => setPage('ai')} className={`px-4 py-2 rounded-lg ${page === 'ai' ? 'bg-green-500 text-white' : 'text-gray-600 hover:bg-gray-100'}`}>AI Insights</button>
          </div>
          <button onClick={() => setCurrentUser(null)} className="text-gray-600 hover:text-gray-800">Logout</button>
        </div>
      </nav>

      <div className="max-w-7xl mx-auto p-6">
        {/* Dashboard Page */}
        {page === 'dashboard' && (
          <div>
            <h2 className="text-3xl font-bold text-gray-800 mb-6">💰 My Money Dashboard</h2>
            
            {/* KPI Cards */}
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
              <div className="bg-white rounded-xl shadow-md p-6">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-600">Monthly Income</span>
                  <DollarSign className="text-green-500" />
                </div>
                <p className="text-3xl font-bold text-gray-800">${totalIncome.toFixed(2)}</p>
              </div>
              
              <div className="bg-white rounded-xl shadow-md p-6">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-600">Monthly Expenses</span>
                  <TrendingUp className="text-red-500" />
                </div>
                <p className="text-3xl font-bold text-gray-800">${totalExpenses.toFixed(2)}</p>
              </div>
              
              <div className="bg-white rounded-xl shadow-md p-6">
                <div className="flex items-center justify-between mb-2">
                  <span className="text-gray-600">Savings Rate</span>
                  <PiggyBank className="text-blue-500" />
                </div>
                <p className="text-3xl font-bold text-gray-800">{savingsRate.toFixed(1)}%</p>
              </div>
            </div>

            {/* Savings Goal Progress */}
            {goal && (
              <div className="bg-white rounded-xl shadow-md p-6 mb-8">
                <h3 className="text-xl font-bold text-gray-800 mb-4">Your Progress to Goal</h3>
                <p className="text-gray-600 mb-2">{goal.goal_name}: ${saved.toFixed(2)} / ${goal.target_amount.toFixed(2)}</p>
                <div className="w-full bg-gray-200 rounded-full h-4">
                  <div className="bg-green-500 h-4 rounded-full transition-all" style={{ width: `${progress}%` }}></div>
                </div>
                <p className="text-right text-sm text-gray-600 mt-2">{progress.toFixed(1)}%</p>
              </div>
            )}

            {/* Charts */}
            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div className="bg-white rounded-xl shadow-md p-6">
                <h3 className="text-xl font-bold text-gray-800 mb-4">Expenses by Category</h3>
                <ResponsiveContainer width="100%" height={300}>
                  <PieChart>
                    <Pie data={getExpensesByCategory()} cx="50%" cy="50%" outerRadius={80} fill="#8884d8" dataKey="value" label>
                      {getExpensesByCategory().map((entry, index) => (
                        <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                      ))}
                    </Pie>
                    <Tooltip />
                    <Legend />
                  </PieChart>
                </ResponsiveContainer>
              </div>

              <div className="bg-white rounded-xl shadow-md p-6">
                <h3 className="text-xl font-bold text-gray-800 mb-4">Income vs Expenses</h3>
                <ResponsiveContainer width="100%" height={300}>
                  <LineChart data={getIncomeVsExpenses()}>
                    <CartesianGrid strokeDasharray="3 3" />
                    <XAxis dataKey="month" />
                    <YAxis />
                    <Tooltip />
                    <Legend />
                    <Line type="monotone" dataKey="income" stroke="#2ecc71" strokeWidth={2} />
                    <Line type="monotone" dataKey="expenses" stroke="#e74c3c" strokeWidth={2} />
                  </LineChart>
                </ResponsiveContainer>
              </div>
            </div>
          </div>
        )}

        {/* Income Page */}
        {page === 'income' && (
          <div>
            <h2 className="text-3xl font-bold text-gray-800 mb-6">Income Tracking</h2>
            
            <div className="bg-white rounded-xl shadow-md p-6 mb-6">
              <h3 className="text-xl font-bold text-gray-800 mb-4">Add New Income</h3>
              <form onSubmit={addIncome} className="grid grid-cols-1 md:grid-cols-4 gap-4">
                <input
                  type="text"
                  placeholder="Source (e.g., Job, Freelance)"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={incomeForm.source}
                  onChange={(e) => setIncomeForm({...incomeForm, source: e.target.value})}
                  required
                />
                <input
                  type="number"
                  step="0.01"
                  placeholder="Amount"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={incomeForm.amount}
                  onChange={(e) => setIncomeForm({...incomeForm, amount: e.target.value})}
                  required
                />
                <input
                  type="date"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={incomeForm.date}
                  onChange={(e) => setIncomeForm({...incomeForm, date: e.target.value})}
                  required
                />
                <button type="submit" className="bg-green-500 text-white px-6 py-2 rounded-lg hover:bg-green-600">
                  Add Income
                </button>
              </form>
            </div>

            <div className="bg-white rounded-xl shadow-md p-6">
              <h3 className="text-xl font-bold text-gray-800 mb-4">Income History</h3>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead>
                    <tr className="border-b">
                      <th className="text-left py-3">Date</th>
                      <th className="text-left py-3">Source</th>
                      <th className="text-right py-3">Amount</th>
                    </tr>
                  </thead>
                  <tbody>
                    {income.sort((a, b) => new Date(b.date) - new Date(a.date)).map(i => (
                      <tr key={i.id} className="border-b hover:bg-gray-50">
                        <td className="py-3">{i.date}</td>
                        <td className="py-3">{i.source}</td>
                        <td className="text-right py-3 text-green-600 font-semibold">${i.amount.toFixed(2)}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* Expenses Page */}
        {page === 'expenses' && (
          <div>
            <h2 className="text-3xl font-bold text-gray-800 mb-6">Expense Tracking</h2>
            
            <div className="bg-white rounded-xl shadow-md p-6 mb-6">
              <h3 className="text-xl font-bold text-gray-800 mb-4">Add New Expense</h3>
              <form onSubmit={addExpense} className="grid grid-cols-1 md:grid-cols-4 gap-4">
                <select
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={expenseForm.category}
                  onChange={(e) => setExpenseForm({...expenseForm, category: e.target.value})}
                >
                  {CATEGORIES.map(cat => <option key={cat} value={cat}>{cat}</option>)}
                </select>
                <input
                  type="number"
                  step="0.01"
                  placeholder="Amount"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={expenseForm.amount}
                  onChange={(e) => setExpenseForm({...expenseForm, amount: e.target.value})}
                  required
                />
                <input
                  type="date"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={expenseForm.date}
                  onChange={(e) => setExpenseForm({...expenseForm, date: e.target.value})}
                  required
                />
                <button type="submit" className="bg-green-500 text-white px-6 py-2 rounded-lg hover:bg-green-600">
                  Add Expense
                </button>
              </form>
            </div>

            <div className="bg-white rounded-xl shadow-md p-6">
              <h3 className="text-xl font-bold text-gray-800 mb-4">Expense History</h3>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead>
                    <tr className="border-b">
                      <th className="text-left py-3">Date</th>
                      <th className="text-left py-3">Category</th>
                      <th className="text-right py-3">Amount</th>
                    </tr>
                  </thead>
                  <tbody>
                    {expenses.sort((a, b) => new Date(b.date) - new Date(a.date)).map(e => (
                      <tr key={e.id} className="border-b hover:bg-gray-50">
                        <td className="py-3">{e.date}</td>
                        <td className="py-3">{e.category}</td>
                        <td className="text-right py-3 text-red-600 font-semibold">${e.amount.toFixed(2)}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* Savings Goals Page */}
        {page === 'goals' && (
          <div>
            <h2 className="text-3xl font-bold text-gray-800 mb-6">Savings Goals</h2>
            
            <div className="bg-white rounded-xl shadow-md p-6 mb-6">
              <h3 className="text-xl font-bold text-gray-800 mb-4">Create New Goal</h3>
              <form onSubmit={addGoal} className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <input
                  type="text"
                  placeholder="Goal Name (e.g., Emergency Fund)"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={goalForm.goal_name}
                  onChange={(e) => setGoalForm({...goalForm, goal_name: e.target.value})}
                  required
                />
                <input
                  type="number"
                  step="0.01"
                  placeholder="Target Amount"
                  className="px-4 py-2 border border-gray-300 rounded-lg"
                  value={goalForm.target_amount}
                  onChange={(e) => setGoalForm({...goalForm, target_amount: e.target.value})}
                  required
                />
                <button type="submit" className="bg-green-500 text-white px-6 py-2 rounded-lg hover:bg-green-600">
                  Create Goal
                </button>
              </form>
            </div>

            <div className="space-y-4">
              {savingsGoals.map(g => {
                const saved = totalIncome - totalExpenses;
                const progress = (saved / g.target_amount) * 100;
                return (
                  <div key={g.id} className="bg-white rounded-xl shadow-md p-6">
                    <div className="flex justify-between items-center mb-4">
                      <h3 className="text-xl font-bold text-gray-800">{g.goal_name}</h3>
                      <span className="text-2xl font-bold text-green-500">{Math.min(progress, 100).toFixed(1)}%</span>
                    </div>
                    <p className="text-gray-600 mb-2">${saved.toFixed(2)} / ${g.target_amount.toFixed(2)}</p>
                    <div className="w-full bg-gray-200 rounded-full h-4">
                      <div className="bg-green-500 h-4 rounded-full transition-all" style={{ width: `${Math.min(progress, 100)}%` }}></div>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {/* AI Insights Page */}
        {page === 'ai' && (
          <div>
            <h2 className="text-3xl font-bold text-gray-800 mb-6">AI Financial Coach</h2>
            
            <div className="bg-white rounded-xl shadow-md p-6 mb-6 text-center">
              <Sparkles className="mx-auto mb-4 text-green-500" size={48} />
              <button
                onClick={getAIInsights}
                disabled={aiLoading}
                className="bg-green-500 text-white px-8 py-3 rounded-lg font-semibold hover:bg-green-600 disabled:opacity-50 disabled:cursor-not-allowed"
              >
                {aiLoading ? '🔮 AI is analyzing your finances...' : '🔮 Get AI Recommendations'}
              </button>
            </div>

            {aiInsights && (
              <div className="bg-white rounded-xl shadow-md p-6">
                <h3 className="text-xl font-bold text-gray-800 mb-4">Here's how you can improve your money flow:</h3>
                <div className="prose prose-green max-w-none">
                  {aiInsights.split('\n').map((line, i) => (
                    <p key={i} className="mb-3 text-gray-700">{line}</p>
                  ))}
                </div>
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}