# secure-user-authentication1. Set Up Authentication
1.1 Initialize Backend
mkdir employee-management-backend
cd employee-management-backend
npm init -y
npm install express mongoose bcryptjs jsonwebtoken dotenv cors
1.2 Folder Structure
backend/
├── controllers/
├── models/
├── routes/
├── middleware/
├── app.js
├── .env
1.3 User Model (MongoDB + Mongoose)
// models/User.js
const mongoose = require('mongoose');
const userSchema = new mongoose.Schema({
  username: { type: String, unique: true },
  password: String,
  role: { type: String, default: 'admin' }
});
module.exports = mongoose.model('User', userSchema);
1.4 Auth Routes (Register/Login)
// routes/auth.js
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const router = express.Router();

router.post('/register', async (req, res) => {
  const hashedPassword = await bcrypt.hash(req.body.password, 10);
  const user = new User({ ...req.body, password: hashedPassword });
  await user.save();
  res.status(201).json({ message: 'User created' });
});

router.post('/login', async (req, res) => {
  const user = await User.findOne({ username: req.body.username });
  if (!user || !(await bcrypt.compare(req.body.password, user.password))) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }
  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
  res.json({ token });
});

👤 2. Employee CRUD Backend
2.1 Employee Model
// models/Employee.js
const mongoose = require('mongoose');
const employeeSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  position: String,
  department: String,
  salary: Number,
});
module.exports = mongoose.model('Employee', employeeSchema);
2.2 Employee Routes (CRUD)
// routes/employees.js
const express = require('express');
const Employee = require('../models/Employee');
const verifyToken = require('../middleware/verifyToken');
const router = express.Router();

router.use(verifyToken); // protect all routes

router.get('/', async (req, res) => {
  const employees = await Employee.find();
  res.json(employees);
});

router.post('/', async (req, res) => {
  const newEmp = new Employee(req.body);
  await newEmp.save();
  res.status(201).json(newEmp);
});

router.put('/:id', async (req, res) => {
  const updatedEmp = await Employee.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(updatedEmp);
});

router.delete('/:id', async (req, res) => {
  await Employee.findByIdAndDelete(req.params.id);
  res.json({ message: 'Deleted' });
});
2.3 Middleware (JWT Verification)
// middleware/verifyToken.js
const jwt = require('jsonwebtoken');

module.exports = function (req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(403).json({ message: 'No token' });
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch {
    res.status(401).json({ message: 'Invalid token' });
  }
};
🧠 3. Frontend (React)
3.1 Set up React App
npx create-react-app employee-management-frontend
cd employee-management-frontend
npm install axios react-router-dom
3.2 Pages to Create
Login Page
Dashboard
Add/Edit Employee
Employee List
3.3 Axios Setup
// utils/axios.js
import axios from 'axios';
const instance = axios.create({
  baseURL: 'http://localhost:5000',
});
instance.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = Bearer ${token};
  return config;
});
export default instance;
3.4 Example: Fetch Employees
// pages/EmployeeList.js
import axios from '../utils/axios';
import { useEffect, useState } from 'react';

function EmployeeList() {
  const [employees, setEmployees] = useState([]);

  useEffect(() => {
    axios.get('/employees')
      .then(res => setEmployees(res.data))
      .catch(err => console.error(err));
  }, []);
return (
    <div>
      <h2>Employees</h2>
      <ul>
        {employees.map(emp => (
          <li key={emp._id}>{emp.name} — {emp.email}</li>
        ))}
      </ul>
    </div>
  );
}

🚀 4. Run Everything
Backend
# In /employee-management-backend
node app.js
Frontend
# In /employee-management-frontend
npm start

✅ 5. Extra Suggestions
Use form libraries like react-hook-form.
Use Yup for frontend validation.
Add protected routes using React Router.
Deploy with Vercel (frontend) and Render/Heroku (backend).
