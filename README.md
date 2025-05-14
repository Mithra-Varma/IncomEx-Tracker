# IncomEx-Tracker
import { useState } from 'react'
import { PieChart, Pie, Cell, BarChart, Bar, XAxis, YAxis, Tooltip, Legend, ResponsiveContainer } from 'recharts'
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "/components/ui/card"
import { Button } from "/components/ui/button"
import { Input } from "/components/ui/input"
import { Label } from "/components/ui/label"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "/components/ui/select"
import { Calendar } from "/components/ui/calendar"
import { Plus, ArrowUp, ArrowDown, X, Wallet, TrendingUp, TrendingDown } from 'lucide-react'

const COLORS = {
  income: '#10b981',
  expense: '#ef4444',
  categories: ['#8b5cf6', '#ec4899', '#f59e0b', '#14b8a6', '#3b82f6', '#f97316']
}

const categories = [
  { id: 1, name: 'Salary', type: 'income', icon: <Wallet className="h-4 w-4" /> },
  { id: 2, name: 'Freelance', type: 'income', icon: <TrendingUp className="h-4 w-4" /> },
  { id: 3, name: 'Food', type: 'expense', icon: <TrendingDown className="h-4 w-4" /> },
  { id: 4, name: 'Transport', type: 'expense', icon: <TrendingDown className="h-4 w-4" /> },
  { id: 5, name: 'Shopping', type: 'expense', icon: <TrendingDown className="h-4 w-4" /> },
  { id: 6, name: 'Entertainment', type: 'expense', icon: <TrendingDown className="h-4 w-4" /> },
]

const mockTransactions = [
  { id: 1, type: 'income', category: 'Salary', amount: 3000, date: '2023-06-01' },
  { id: 2, type: 'expense', category: 'Food', amount: 45, date: '2023-06-02' },
  { id: 3, type: 'expense', category: 'Transport', amount: 30, date: '2023-06-03' },
  { id: 4, type: 'expense', category: 'Shopping', amount: 120, date: '2023-06-05' },
  { id: 5, type: 'income', category: 'Freelance', amount: 500, date: '2023-06-10' },
]

export default function IncomExTracker() {
  const [transactions, setTransactions] = useState(mockTransactions)
  const [showAddForm, setShowAddForm] = useState(false)
  const [formData, setFormData] = useState({
    type: 'expense',
    category: '',
    amount: '',
    date: new Date(),
    notes: ''
  })

  // Calculate totals
  const totalIncome = transactions
    .filter(t => t.type === 'income')
    .reduce((sum, t) => sum + t.amount, 0)

  const totalExpenses = transactions
    .filter(t => t.type === 'expense')
    .reduce((sum, t) => sum + t.amount, 0)

  const balance = totalIncome - totalExpenses

  // Prepare data for charts
  const expenseByCategory = categories
    .filter(c => c.type === 'expense')
    .map(category => {
      const total = transactions
        .filter(t => t.category === category.name)
        .reduce((sum, t) => sum + t.amount, 0)
      return {
        name: category.name,
        value: total,
        icon: category.icon
      }
    })
    .filter(item => item.value > 0)

  const monthlyData = [
    { name: 'Income', value: totalIncome, fill: COLORS.income },
    { name: 'Expenses', value: totalExpenses, fill: COLORS.expense }
  ]

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target
    setFormData(prev => ({ ...prev, [name]: value }))
  }

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    const newTransaction = {
      id: transactions.length + 1,
      type: formData.type,
      category: formData.category,
      amount: Number(formData.amount),
      date: formData.date.toISOString().split('T')[0],
      notes: formData.notes
    }
    setTransactions([...transactions, newTransaction])
    setFormData({
      type: 'expense',
      category: '',
      amount: '',
      date: new Date(),
      notes: ''
    })
    setShowAddForm(false)
  }

  const formatDate = (dateString: string) => {
    const options: Intl.DateTimeFormatOptions = { month: 'short', day: 'numeric' }
    return new Date(dateString).toLocaleDateString(undefined, options)
  }

  const getCategoryColor = (categoryName: string) => {
    const index = categories.findIndex(c => c.name === categoryName)
    return COLORS.categories[index % COLORS.categories.length]
  }

  return (
    <div className="min-h-screen bg-gradient-to-b from-violet-50 to-white">
      <div className="container mx-auto px-4 py-8 max-w-6xl">
        {/* Header */}
        <header className="mb-8 text-center">
          <h1 className="text-4xl font-bold text-violet-800 mb-2">IncomEx</h1>
          <p className="text-violet-600 text-lg">Visualize your financial health</p>
        </header>

        {/* Dashboard */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
          {/* Balance Card */}
          <Card className="bg-white shadow-sm border-0">
            <CardHeader className="pb-2">
              <CardDescription className="text-sm text-violet-600 flex items-center">
                <Wallet className="h-4 w-4 mr-2" /> Balance
              </CardDescription>
              <CardTitle className={`text-2xl ${balance >= 0 ? 'text-emerald-600' : 'text-rose-600'}`}>
                ${balance.toFixed(2)}
              </CardTitle>
            </CardHeader>
            <CardContent>
              <div className="flex items-center text-sm text-violet-500">
                {balance >= 0 ? (
                  <ArrowUp className="h-4 w-4 mr-1 text-emerald-500" />
                ) : (
                  <ArrowDown className="h-4 w-4 mr-1 text-rose-500" />
                )}
                {totalIncome > 0 ? Math.abs(balance/totalIncome*100).toFixed(1) : '0'}% {balance >= 0 ? 'surplus' : 'deficit'}
              </div>
            </CardContent>
          </Card>

          {/* Income Card */}
          <Card className="bg-white shadow-sm border-0">
            <CardHeader className="pb-2">
              <CardDescription className="text-sm text-violet-600 flex items-center">
                <TrendingUp className="h-4 w-4 mr-2" /> Income
              </CardDescription>
              <CardTitle className="text-2xl text-emerald-600">${totalIncome.toFixed(2)}</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="flex items-center text-sm text-violet-500">
                <ArrowUp className="h-4 w-4 mr-1 text-emerald-500" />
                {transactions.filter(t => t.type === 'income').length} transactions
              </div>
            </CardContent>
          </Card>

          {/* Expenses Card */}
          <Card className="bg-white shadow-sm border-0">
            <CardHeader className="pb-2">
              <CardDescription className="text-sm text-violet-600 flex items-center">
                <TrendingDown className="h-4 w-4 mr-2" /> Expenses
              </CardDescription>
              <CardTitle className="text-2xl text-rose-600">${totalExpenses.toFixed(2)}</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="flex items-center text-sm text-violet-500">
                <ArrowDown className="h-4 w-4 mr-1 text-rose-500" />
                {transactions.filter(t => t.type === 'expense').length} transactions
              </div>
            </CardContent>
          </Card>
        </div>

        {/* Visualization Section */}
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
          {/* Income vs Expenses Bar Chart */}
          <Card className="bg-white shadow-sm border-0">
            <CardHeader>
              <CardTitle className="text-lg text-violet-800">Income vs Expenses</CardTitle>
            </CardHeader>
            <CardContent className="h-64">
              <ResponsiveContainer width="100%" height="100%">
                <BarChart data={monthlyData}>
                  <XAxis dataKey="name" />
                  <YAxis />
                  <Tooltip 
                    formatter={(value) => [`$${value}`, 'Amount']}
                    labelFormatter={(label) => `${label}`}
                  />
                  <Bar dataKey="value" radius={[4, 4, 0, 0]}>
                    {monthlyData.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={entry.fill} />
                    ))}
                  </Bar>
                </BarChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>

          {/* Expense Breakdown Pie Chart */}
          {expenseByCategory.length > 0 && (
            <Card className="bg-white shadow-sm border-0">
              <CardHeader>
                <CardTitle className="text-lg text-violet-800">Expense Breakdown</CardTitle>
              </CardHeader>
              <CardContent className="h-64">
                <ResponsiveContainer width="100%" height="100%">
                  <PieChart>
                    <Pie
                      data={expenseByCategory}
                      cx="50%"
                      cy="50%"
                      labelLine={false}
                      outerRadius={80}
                      fill="#8884d8"
                      dataKey="value"
                      label={({ name, percent }) => `${name} ${(percent * 100).toFixed(0)}%`}
                    >
                      {expenseByCategory.map((entry, index) => (
                        <Cell key={`cell-${index}`} fill={getCategoryColor(entry.name)} />
                      ))}
                    </Pie>
                    <Tooltip 
                      formatter={(value) => [`$${value}`, 'Amount']}
                    />
                    <Legend />
                  </PieChart>
                </ResponsiveContainer>
              </CardContent>
            </Card>
          )}
        </div>

        {/* Recent Transactions */}
        <Card className="mb-8 bg-white shadow-sm border-0">
          <CardHeader>
            <CardTitle className="text-xl text-violet-800">Recent Transactions</CardTitle>
          </CardHeader>
          <CardContent>
            {transactions.length === 0 ? (
              <div className="text-center py-8 text-violet-400">
                <p>No transactions yet</p>
                <p className="text-sm mt-2">Add your first transaction to get started</p>
              </div>
            ) : (
              <div className="space-y-3">
                {transactions.slice().reverse().map(transaction => {
                  const categoryInfo = categories.find(c => c.name === transaction.category)
                  return (
                    <div 
                      key={transaction.id} 
                      className="flex items-center justify-between p-3 rounded-lg hover:bg-violet-50 transition-colors"
                    >
                      <div className="flex items-center space-x-3">
                        <div 
                          className="p-2 rounded-full"
                          style={{ 
                            backgroundColor: transaction.type === 'income' 
                              ? COLORS.income + '20' 
                              : getCategoryColor(transaction.category) + '20',
                            color: transaction.type === 'income' 
                              ? COLORS.income 
                              : getCategoryColor(transaction.category)
                          }}
                        >
                          {categoryInfo?.icon || 
                            (transaction.type === 'income' ? 
                              <ArrowUp className="h-5 w-5" /> : 
                              <ArrowDown className="h-5 w-5" />)}
                        </div>
                        <div>
                          <p className="font-medium text-violet-800">{transaction.category}</p>
                          <p className="text-sm text-violet-400">{formatDate(transaction.date)}</p>
                          {transaction.notes && (
                            <p className="text-xs text-violet-300 mt-1">{transaction.notes}</p>
                          )}
                        </div>
                      </div>
                      <p 
                        className="font-bold" 
                        style={{
                          color: transaction.type === 'income' ? COLORS.income : COLORS.expense
                        }}
                      >
                        {transaction.type === 'income' ? '+' : '-'}${transaction.amount.toFixed(2)}
                      </p>
                    </div>
                  )
                })}
              </div>
            )}
          </CardContent>
        </Card>

        {/* Add Transaction Button */}
        <div className="fixed bottom-8 right-8">
          <Button
            onClick={() => setShowAddForm(true)}
            className="rounded-full h-14 w-14 shadow-lg bg-violet-600 hover:bg-violet-700 transition-transform hover:scale-105"
          >
            <Plus className="h-6 w-6" />
          </Button>
        </div>

        {/* Add Transaction Form Modal */}
        {showAddForm && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
            <Card className="w-full max-w-md bg-white shadow-xl">
              <CardHeader className="relative">
                <CardTitle className="text-violet-800">Add Transaction</CardTitle>
                <Button 
                  variant="ghost" 
                  size="icon" 
                  className="absolute right-4 top-4 text-violet-400 hover:text-violet-600"
                  onClick={() => setShowAddForm(false)}
                >
                  <X className="h-5 w-5" />
                </Button>
              </CardHeader>
              <CardContent>
                <form onSubmit={handleSubmit} className="space-y-4">
                  <div>
                    <Label className="text-violet-700">Type</Label>
                    <div className="flex space-x-2 mt-2">
                      <Button
                        type="button"
                        variant={formData.type === 'income' ? 'default' : 'outline'}
                        className={`flex-1 ${formData.type === 'income' ? 'bg-emerald-600 hover:bg-emerald-700' : ''}`}
                        onClick={() => setFormData({ ...formData, type: 'income' })}
                      >
                        <ArrowUp className="h-4 w-4 mr-2" />
                        Income
                      </Button>
                      <Button
                        type="button"
                        variant={formData.type === 'expense' ? 'default' : 'outline'}
                        className={`flex-1 ${formData.type === 'expense' ? 'bg-rose-600 hover:bg-rose-700' : ''}`}
                        onClick={() => setFormData({ ...formData, type: 'expense' })}
                      >
                        <ArrowDown className="h-4 w-4 mr-2" />
                        Expense
                      </Button>
                    </div>
                  </div>

                  <div>
                    <Label className="text-violet-700">Category</Label>
                    <Select
                      value={formData.category}
                      onValueChange={(value) => setFormData({ ...formData, category: value })}
                    >
                      <SelectTrigger className="w-full">
                        <SelectValue placeholder="Select category" />
                      </SelectTrigger>
                      <SelectContent>
                        {categories
                          .filter(cat => cat.type === formData.type)
                          .map(category => (
                            <SelectItem 
                              key={category.id} 
                              value={category.name}
                              className="flex items-center"
                            >
                              <span className="mr-2">{category.icon}</span>
                              {category.name}
                            </SelectItem>
                          ))}
                      </SelectContent>
                    </Select>
                  </div>

                  <div>
                    <Label className="text-violet-700">Amount</Label>
                    <div className="relative">
                      <span className="absolute left-3 top-1/2 transform -translate-y-1/2 text-violet-400">$</span>
                      <Input
                        type="number"
                        name="amount"
                        value={formData.amount}
                        onChange={handleInputChange}
                        placeholder="0.00"
                        required
                        className="pl-8 text-lg"
                      />
                    </div>
                  </div>

                  <div>
                    <Label className="text-violet-700">Date</Label>
                    <Calendar
                      mode="single"
                      selected={formData.date}
                      onSelect={(date) => date && setFormData({ ...formData, date })}
                      className="rounded-md border mt-2"
                    />
                  </div>

                  <div>
                    <Label className="text-violet-700">Notes (Optional)</Label>
                    <Input
                      name="notes"
                      value={formData.notes}
                      onChange={handleInputChange}
                      placeholder="Add a note..."
                    />
                  </div>

                  <div className="flex justify-end space-x-2 pt-4">
                    <Button
                      type="button"
                      variant="outline"
                      className="text-violet-600 border-violet-300 hover:bg-violet-50"
                      onClick={() => setShowAddForm(false)}
                    >
                      Cancel
                    </Button>
                    <Button 
                      type="submit" 
                      className="bg-violet-600 hover:bg-violet-700"
                    >
                      Add Transaction
                    </Button>
                  </div>
                </form>
              </CardContent>
            </Card>
          </div>
        )}
      </div>
    </div>
  )
}
