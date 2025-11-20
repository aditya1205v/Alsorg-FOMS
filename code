import React, { useState, useEffect, useCallback, useMemo } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signOut } from 'firebase/auth';
import { getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where, getDocs, serverTimestamp, getDocFromCache, setLogLevel } from 'firebase/firestore';
import { Menu, Home, Calendar, Users, Briefcase, DollarSign, List, FileText, X, Menu as MenuIcon, LogOut, Clock, Layers, TrendingUp, Folder, AlertTriangle, ChevronRight, CheckCircle, UploadCloud } from 'lucide-react';

// =========================================================================
// !!! CRITICAL DEPLOYMENT SECTION - PASTE YOUR REAL FIREBASE CONFIG HERE !!!
// =========================================================================
// This section replaces the special Canvas variables (which don't exist outside this environment).
// You MUST replace the placeholder strings with the actual values from your Firebase Console.

const LIVE_APP_ID = "alsorg-foms-live-prod"; // A unique identifier for your app instance
const LIVE_FIREBASE_CONFIG = {
    // ----------------------------------------------------------------------
    // PASTE YOUR REAL FIREBASE CONFIGURATION BELOW. DO NOT use the quotes "" 
    // around numbers if Firebase gave you numbers.
    // ----------------------------------------------------------------------
    apiKey: "AIzaSyBR9I6OWurmcAWJWWpXMEyvhNojeU5iD2Q", 
    authDomain: "alsorg-foms-live.firebaseapp.com", 
    projectId: "alsorg-foms-live", 
    storageBucket: "alsorg-foms-live.firebasestorage.app", 
    messagingSenderId: "714560336226",
    appId: "1:714560336226:web:550b1930f80bb193ed3c3b" 
    // ----------------------------------------------------------------------
};

// Use the live configuration for deployment
const appId = LIVE_APP_ID;
const firebaseConfig = LIVE_FIREBASE_CONFIG;
// =========================================================================
// !!! END OF CRITICAL DEPLOYMENT SECTION !!!
// =========================================================================


// -------------------------
// Helper Functions
// -------------------------

const generateCollectionPath = (collectionName, userId, isPublic = false) => {
    // Note: The /public/data/ path is used for public (shared) documents.
    if (isPublic) {
        return `artifacts/${appId}/public/data/${collectionName}`;
    }
    // The default path is private for the user
    return `artifacts/${appId}/users/${userId}/${collectionName}`;
};

// Error Modal Component (replaces alert() and confirm())
const Modal = ({ title, message, isOpen, onClose }) => {
    if (!isOpen) return null;
    return (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
            <div className="bg-white rounded-lg shadow-2xl p-6 w-full max-w-sm transform transition-all duration-300">
                <h3 className="text-xl font-bold text-red-600 border-b pb-2 mb-4">{title}</h3>
                <p className="text-gray-700 mb-6">{message}</p>
                <div className="flex justify-end">
                    <button
                        onClick={onClose}
                        className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition duration-150 shadow-md"
                    >
                        Close
                    </button>
                </div>
            </div>
        </div>
    );
};

// -------------------------
// Navigation Components
// -------------------------

const NavItem = ({ icon: Icon, title, active, onClick }) => (
    <div
        className={`flex items-center p-3 rounded-xl cursor-pointer transition duration-200 ${
            active
                ? 'bg-blue-600 text-white shadow-lg font-semibold'
                : 'text-gray-700 hover:bg-blue-100'
        }`}
        onClick={onClick}
    >
        <Icon className="w-5 h-5 mr-3" />
        <span className="hidden md:inline">{title}</span>
    </div>
);

const NavFooter = ({ userId, onSignOut }) => (
    <div className="p-4 border-t border-gray-200 mt-auto">
        <p className="text-xs text-gray-500 mb-2 truncate">
            User ID: **{userId || 'N/A'}**
        </p>
        <button
            onClick={onSignOut}
            className="w-full flex items-center justify-center p-2 text-sm bg-red-500 text-white rounded-xl hover:bg-red-600 transition duration-150 shadow-md"
        >
            <LogOut className="w-4 h-4 mr-2" />
            Sign Out
        </button>
    </div>
);

// -------------------------
// Data Structures (Shared)
// -------------------------

const UserRoles = {
    ADMIN: 'Administrator',
    SUPERVISOR: 'Supervisor',
    WORKER: 'Worker',
};

const ExpenseCategories = [
    'Food', 'Travel', 'Material Purchase', 'Rental Stay', 'Utilities', 'Miscellaneous'
];

const IssuePriorities = [
    'Low', 'Medium', 'High', 'Critical'
];

const ProjectStatuses = [
    'Active', 'On Hold', 'Completed', 'Planning'
];

// -------------------------
// Dashboard Component
// -------------------------

const Dashboard = ({ projects, expenses, attendance, userId, userRole }) => {
    const totalProjects = projects.length;
    const activeProjects = projects.filter(p => p.status === 'Active').length;
    const todayExpenses = expenses.filter(e => new Date(e.date).toDateString() === new Date().toDateString());
    const totalExpenses = expenses.reduce((sum, e) => sum + e.amount, 0);

    const projectCompletion = useMemo(() => {
        if (!totalProjects) return 0;
        const completedCount = projects.filter(p => p.status === 'Completed').length;
        return Math.round((completedCount / totalProjects) * 100);
    }, [projects, totalProjects]);

    // Simple attendance summary
    const presentCount = attendance.filter(a => new Date(a.clockIn).toDateString() === new Date().toDateString()).length;

    const Card = ({ title, value, icon: Icon, color }) => (
        <div className={`bg-white p-6 rounded-2xl shadow-xl transition transform hover:scale-[1.02] duration-300 border-l-4 ${color}`}>
            <div className="flex items-center justify-between">
                <div>
                    <p className="text-sm font-medium text-gray-500">{title}</p>
                    <h4 className="text-3xl font-bold text-gray-800 mt-1">
                        {title.includes('Expense') ? `₹${value.toLocaleString()}` : value}
                    </h4>
                </div>
                <Icon className={`w-8 h-8 opacity-50 ${color.replace('border-', 'text-')}`} />
            </div>
        </div>
    );

    return (
        <div className="space-y-8 p-4 md:p-8">
            <h2 className="text-3xl font-extrabold text-gray-800">Operations Dashboard</h2>

            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
                <Card
                    title="Total Projects"
                    value={totalProjects}
                    icon={Folder}
                    color="border-blue-500"
                />
                <Card
                    title="Active Sites"
                    value={activeProjects}
                    icon={Layers}
                    color="border-green-500"
                />
                <Card
                    title="Total Expenses YTD"
                    value={totalExpenses}
                    icon={DollarSign}
                    color="border-yellow-500"
                />
                <Card
                    title="Today's Check-ins"
                    value={presentCount}
                    icon={Clock}
                    color="border-purple-500"
                />
            </div>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
                <div className="lg:col-span-2 bg-white p-6 rounded-2xl shadow-xl">
                    <h3 className="text-xl font-bold mb-4 text-gray-800">Project Completion Status</h3>
                    <div className="space-y-4">
                        {projects.map(project => (
                            <ProjectStatusItem key={project.id} project={project} />
                        ))}
                    </div>
                </div>

                <div className="bg-white p-6 rounded-2xl shadow-xl">
                    <h3 className="text-xl font-bold mb-4 text-gray-800">Today's Financial Activity</h3>
                    <p className="text-4xl font-bold text-green-600 mb-6">
                        ₹{todayExpenses.reduce((sum, e) => sum + e.amount, 0).toLocaleString()}
                    </p>
                    <p className="text-gray-600">
                        {todayExpenses.length} expense entries submitted today by supervisors.
                    </p>
                    <button className="mt-4 w-full bg-blue-500 text-white p-3 rounded-xl hover:bg-blue-600 transition">
                        View Detailed Expense Report
                    </button>
                </div>
            </div>
        </div>
    );
};

const ProjectStatusItem = ({ project }) => {
    const progress = project.completedWorkUnits && project.totalWorkUnits
        ? Math.round((project.completedWorkUnits / project.totalWorkUnits) * 100)
        : 0;

    return (
        <div className="border-b pb-3">
            <div className="flex justify-between items-center mb-1">
                <span className="font-semibold text-gray-700 truncate">{project.name}</span>
                <span className={`text-sm font-medium ${progress === 100 ? 'text-green-500' : 'text-blue-500'}`}>{progress}%</span>
            </div>
            <div className="w-full bg-gray-200 rounded-full h-2.5">
                <div
                    className="h-2.5 rounded-full transition-all duration-500"
                    style={{ width: `${Math.min(100, progress)}%`, backgroundColor: progress === 100 ? '#10B981' : '#3B82F6' }}
                ></div>
            </div>
        </div>
    );
};


// -------------------------
// Attendance Component
// -------------------------

const Attendance = ({ db, userId, projects, setIsModalOpen, setModalMessage, setModalTitle }) => {
    const [selectedSite, setSelectedSite] = useState('');
    const [status, setStatus] = useState('Clock In');
    const [loading, setLoading] = useState(false);
    const [currentLocation, setCurrentLocation] = useState(null);
    const [attendanceHistory, setAttendanceHistory] = useState([]);
    const [message, setMessage] = useState('');

    const attendanceCollectionPath = generateCollectionPath('attendance', userId, true);

    useEffect(() => {
        // Fetch Location (Mocked for safety/iframe limitations)
        setCurrentLocation({ lat: 28.5355, lon: 77.3910, mock: true }); // Mocked location (Noida/Delhi region)

        if (!db || !userId) return;

        // Fetch Attendance History
        const q = query(collection(db, attendanceCollectionPath));
        const unsubscribe = onSnapshot(q, (snapshot) => {
            const history = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setAttendanceHistory(history.sort((a, b) => (b.clockIn || 0) - (a.clockIn || 0)));
        }, (error) => {
            console.error("Error fetching attendance history:", error);
        });

        return () => unsubscribe();
    }, [db, userId, attendanceCollectionPath]);

    const handleAction = async () => {
        if (!selectedSite || !currentLocation) {
            setMessage('Please select a site and ensure location services are available.');
            return;
        }

        // Mock Geo-fencing check (always passes in this mock env)
        const isWithinSite = true;

        if (!isWithinSite) {
            setMessage('Error: You are not within the geo-fenced area of the selected site.');
            return;
        }

        setLoading(true);
        setMessage('');

        try {
            if (status === 'Clock In') {
                await addDoc(collection(db, attendanceCollectionPath), {
                    siteId: selectedSite,
                    siteName: projects.find(p => p.id === selectedSite)?.name || 'Unknown Site',
                    userId: userId,
                    clockIn: serverTimestamp(),
                    locationIn: currentLocation,
                    status: 'Active',
                    date: new Date().toISOString().substring(0, 10),
                    // Mock: Selfie/Photo upload would go here (storage URL)
                    selfieUrl: `https://placehold.co/100x100/F472B6/ffffff?text=Selfie+for+${selectedSite}`
                });
                setStatus('Clock Out');
                setModalTitle("Success");
                setModalMessage("Clock-in successful. Happy working!");
                setIsModalOpen(true);

            } else { // Clock Out
                // Find the latest active check-in for the user on this date
                const latestActive = attendanceHistory.find(
                    a => a.siteId === selectedSite && a.status === 'Active'
                );

                if (latestActive) {
                    const docRef = doc(db, attendanceCollectionPath, latestActive.id);
                    await updateDoc(docRef, {
                        clockOut: serverTimestamp(),
                        locationOut: currentLocation,
                        status: 'Completed'
                    });
                    setStatus('Clock In');
                    setModalTitle("Success");
                    setModalMessage("Clock-out successful. Have a good rest!");
                    setIsModalOpen(true);
                } else {
                    setMessage("Error: No active check-in found to clock out.");
                }
            }
        } catch (e) {
            console.error("Error handling attendance action: ", e);
            setModalTitle("Operation Failed");
            setModalMessage("Could not process your request. Check console for details.");
            setIsModalOpen(true);
        } finally {
            setLoading(false);
        }
    };

    const getLocationDisplay = () => {
        if (currentLocation && currentLocation.mock) {
            return "Location (Mocked): Lat: " + currentLocation.lat + ", Lon: " + currentLocation.lon;
        }
        return currentLocation ? `Lat: ${currentLocation.lat.toFixed(4)}, Lon: ${currentLocation.lon.toFixed(4)}` : 'Fetching Location...';
    };

    return (
        <div className="p-4 md:p-8 space-y-6">
            <h2 className="text-3xl font-extrabold text-gray-800">Daily Attendance</h2>

            <div className="bg-white p-6 rounded-2xl shadow-xl space-y-4 max-w-lg mx-auto">
                <p className="text-lg font-semibold text-gray-700">Select Site & Take Action</p>

                <select
                    value={selectedSite}
                    onChange={(e) => {
                        setSelectedSite(e.target.value);
                        setMessage('');
                    }}
                    className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                >
                    <option value="">-- Select Project Site --</option>
                    {projects.filter(p => p.status === 'Active').map(p => (
                        <option key={p.id} value={p.id}>{p.name}</option>
                    ))}
                </select>

                <div className="text-sm text-gray-600 border p-3 rounded-xl bg-gray-50">
                    <p className="font-medium">GPS Status: Ready (Mocked)</p>
                    <p className="truncate">{getLocationDisplay()}</p>
                    <p className="text-xs text-red-500 mt-1">
                        **NOTE:** Live deployment requires real GPS access and camera for selfie.
                    </p>
                </div>

                <button
                    onClick={handleAction}
                    disabled={!selectedSite || loading}
                    className={`w-full p-4 rounded-xl text-white font-bold transition duration-300 ${
                        loading ? 'bg-gray-400' : status === 'Clock In' ? 'bg-green-600 hover:bg-green-700 shadow-md' : 'bg-red-600 hover:bg-red-700 shadow-md'
                    }`}
                >
                    {loading ? 'Processing...' : status}
                </button>
                {message && <p className="text-red-500 text-center mt-2">{message}</p>}
            </div>

            <AttendanceHistoryTable history={attendanceHistory} projects={projects} />
        </div>
    );
};

const AttendanceHistoryTable = ({ history, projects }) => {
    const getSiteName = (id) => projects.find(p => p.id === id)?.name || 'N/A';
    const formatTime = (timestamp) => timestamp ? new Date(timestamp.seconds * 1000).toLocaleTimeString() : '---';
    const formatDate = (timestamp) => timestamp ? new Date(timestamp.seconds * 1000).toLocaleDateString() : '---';

    return (
        <div className="bg-white p-6 rounded-2xl shadow-xl mt-8">
            <h3 className="text-xl font-bold mb-4 text-gray-800">Recent History</h3>
            <div className="overflow-x-auto">
                <table className="min-w-full divide-y divide-gray-200">
                    <thead>
                        <tr className="bg-gray-50">
                            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Date</th>
                            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Site</th>
                            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Clock In</th>
                            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Clock Out</th>
                            <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                        </tr>
                    </thead>
                    <tbody className="bg-white divide-y divide-gray-200">
                        {history.slice(0, 10).map(item => (
                            <tr key={item.id} className="hover:bg-gray-50">
                                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{formatDate(item.clockIn)}</td>
                                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">{item.siteName}</td>
                                <td className="px-6 py-4 whitespace-nowrap text-sm text-green-600 font-semibold">{formatTime(item.clockIn)}</td>
                                <td className="px-6 py-4 whitespace-nowrap text-sm">{item.clockOut ? formatTime(item.clockOut) : <span className="text-yellow-500">Active</span>}</td>
                                <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                    <span className={`px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${
                                        item.status === 'Active' ? 'bg-yellow-100 text-yellow-800' : 'bg-green-100 text-green-800'
                                    }`}>
                                        {item.status}
                                    </span>
                                </td>
                            </tr>
                        ))}
                    </tbody>
                </table>
            </div>
            {history.length === 0 && <p className="text-center py-4 text-gray-500">No attendance records found.</p>}
        </div>
    );
};

// -------------------------
// Expenses Component
// -------------------------

const Expenses = ({ db, userId, projects, setIsModalOpen, setModalMessage, setModalTitle }) => {
    const [selectedSite, setSelectedSite] = useState('');
    const [category, setCategory] = useState(ExpenseCategories[0]);
    const [amount, setAmount] = useState('');
    const [description, setDescription] = useState('');
    const [loading, setLoading] = useState(false);
    const [expenses, setExpenses] = useState([]);

    const expenseCollectionPath = generateCollectionPath('expenses', userId, true);

    useEffect(() => {
        if (!db || !userId) return;

        const q = query(collection(db, expenseCollectionPath));
        const unsubscribe = onSnapshot(q, (snapshot) => {
            const list = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setExpenses(list.sort((a, b) => (b.timestamp || 0) - (a.timestamp || 0)));
        }, (error) => {
            console.error("Error fetching expenses:", error);
        });

        return () => unsubscribe();
    }, [db, userId, expenseCollectionPath]);

    const handleAddExpense = async (e) => {
        e.preventDefault();
        const expenseAmount = parseFloat(amount);

        if (!selectedSite || expenseAmount <= 0 || !description) {
            setModalTitle("Validation Error");
            setModalMessage("Please fill all required fields correctly (Site, Amount > 0, Description).");
            setIsModalOpen(true);
            return;
        }

        setLoading(true);

        try {
            await addDoc(collection(db, expenseCollectionPath), {
                siteId: selectedSite,
                siteName: projects.find(p => p.id === selectedSite)?.name || 'Unknown Site',
                userId: userId,
                category,
                amount: expenseAmount,
                description,
                receiptUrl: `https://placehold.co/150x150/06B6D4/ffffff?text=${category}+Receipt`, // Mock Receipt
                status: 'Pending', // Initial status
                timestamp: serverTimestamp(),
                date: new Date().toISOString().substring(0, 10),
            });

            setModalTitle("Success");
            setModalMessage("Expense submitted successfully for review!");
            setIsModalOpen(true);
            // Reset form
            setAmount('');
            setDescription('');
            setSelectedSite('');
            setCategory(ExpenseCategories[0]);

        } catch (error) {
            console.error("Error adding expense: ", error);
            setModalTitle("Operation Failed");
            setModalMessage("Could not submit expense. Please try again.");
            setIsModalOpen(true);
        } finally {
            setLoading(false);
        }
    };

    const getStatusStyles = (status) => {
        switch (status) {
            case 'Approved': return 'bg-green-100 text-green-800';
            case 'Rejected': return 'bg-red-100 text-red-800';
            case 'Pending':
            default: return 'bg-yellow-100 text-yellow-800';
        }
    };

    const totalSubmitted = expenses.reduce((sum, e) => sum + e.amount, 0);

    return (
        <div className="p-4 md:p-8 space-y-8">
            <h2 className="text-3xl font-extrabold text-gray-800">Expense Management</h2>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
                {/* Expense Submission Form */}
                <div className="bg-white p-6 rounded-2xl shadow-xl lg:col-span-1 h-fit">
                    <h3 className="text-xl font-bold mb-6 text-gray-800 border-b pb-3">Submit New Expense</h3>
                    <form onSubmit={handleAddExpense} className="space-y-4">
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Project Site</label>
                            <select
                                value={selectedSite}
                                onChange={(e) => setSelectedSite(e.target.value)}
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                required
                            >
                                <option value="">-- Select Project Site --</option>
                                {projects.filter(p => p.status === 'Active').map(p => (
                                    <option key={p.id} value={p.id}>{p.name}</option>
                                ))}
                            </select>
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Category</label>
                            <select
                                value={category}
                                onChange={(e) => setCategory(e.target.value)}
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                            >
                                {ExpenseCategories.map(cat => (
                                    <option key={cat} value={cat}>{cat}</option>
                                ))}
                            </select>
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Amount (₹)</label>
                            <input
                                type="number"
                                step="0.01"
                                value={amount}
                                onChange={(e) => setAmount(e.target.value)}
                                placeholder="e.g., 550.00"
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                required
                            />
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Description/Purpose</label>
                            <textarea
                                value={description}
                                onChange={(e) => setDescription(e.target.value)}
                                placeholder="Briefly explain the purpose of this expense (e.g., lunch for 5 workers)"
                                rows="3"
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                required
                            ></textarea>
                        </div>
                        <div className="flex items-center justify-between p-3 border rounded-xl bg-gray-50">
                             <UploadCloud className="w-5 h-5 text-blue-500 mr-2" />
                             <span className="text-sm text-gray-700 flex-grow">Receipt Image Upload (Mocked)</span>
                             <button type="button" className="text-xs text-blue-600 hover:text-blue-800 font-medium">Select File</button>
                        </div>
                        <button
                            type="submit"
                            disabled={loading}
                            className="w-full p-3 mt-4 bg-blue-600 text-white font-bold rounded-xl hover:bg-blue-700 transition duration-300 shadow-lg disabled:bg-gray-400 flex items-center justify-center"
                        >
                            {loading ? 'Submitting...' : 'Submit Expense'}
                        </button>
                    </form>
                </div>

                {/* Expense History Table */}
                <div className="lg:col-span-2 bg-white p-6 rounded-2xl shadow-xl">
                    <h3 className="text-xl font-bold mb-4 text-gray-800 border-b pb-3">Expense History</h3>
                    <div className="flex justify-between items-center mb-4 text-sm font-medium">
                        <p className="text-gray-600">Total Submitted (All Time):</p>
                        <p className="text-2xl font-bold text-red-600">₹{totalSubmitted.toLocaleString()}</p>
                    </div>
                    <div className="overflow-x-auto">
                        <table className="min-w-full divide-y divide-gray-200">
                            <thead>
                                <tr className="bg-gray-50">
                                    <th className="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Date</th>
                                    <th className="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Site</th>
                                    <th className="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Category</th>
                                    <th className="px-3 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">Amount (₹)</th>
                                    <th className="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                                </tr>
                            </thead>
                            <tbody className="bg-white divide-y divide-gray-200">
                                {expenses.slice(0, 10).map(e => (
                                    <tr key={e.id} className="hover:bg-gray-50">
                                        <td className="px-3 py-3 whitespace-nowrap text-sm text-gray-500">{new Date(e.timestamp?.seconds * 1000).toLocaleDateString()}</td>
                                        <td className="px-3 py-3 whitespace-nowrap text-sm font-medium text-gray-900 truncate max-w-xs">{e.siteName}</td>
                                        <td className="px-3 py-3 whitespace-nowrap text-sm text-gray-500">{e.category}</td>
                                        <td className="px-3 py-3 whitespace-nowrap text-sm text-right font-bold text-red-600">₹{e.amount.toLocaleString()}</td>
                                        <td className="px-3 py-3 whitespace-nowrap">
                                            <span className={`px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusStyles(e.status)}`}>
                                                {e.status}
                                            </span>
                                        </td>
                                    </tr>
                                ))}
                            </tbody>
                        </table>
                    </div>
                    {expenses.length === 0 && <p className="text-center py-4 text-gray-500">No expense records found.</p>}
                </div>
            </div>
        </div>
    );
};

// -------------------------
// Issues Component
// -------------------------

const Issues = ({ db, userId, projects, setIsModalOpen, setModalMessage, setModalTitle, userRole }) => {
    const [selectedSite, setSelectedSite] = useState('');
    const [title, setTitle] = useState('');
    const [description, setDescription] = useState('');
    const [priority, setPriority] = useState(IssuePriorities[0]);
    const [issues, setIssues] = useState([]);
    const [loading, setLoading] = useState(false);

    const issueCollectionPath = generateCollectionPath('issues', userId, true);

    useEffect(() => {
        if (!db || !userId) return;

        const q = query(collection(db, issueCollectionPath));
        const unsubscribe = onSnapshot(q, (snapshot) => {
            const list = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setIssues(list.sort((a, b) => (b.timestamp || 0) - (a.timestamp || 0)));
        }, (error) => {
            console.error("Error fetching issues:", error);
        });

        return () => unsubscribe();
    }, [db, userId, issueCollectionPath]);

    const handleAddIssue = async (e) => {
        e.preventDefault();
        if (!selectedSite || !title || !description) {
            setModalTitle("Validation Error");
            setModalMessage("Please fill all required fields (Site, Title, Description).");
            setIsModalOpen(true);
            return;
        }

        setLoading(true);

        try {
            await addDoc(collection(db, issueCollectionPath), {
                siteId: selectedSite,
                siteName: projects.find(p => p.id === selectedSite)?.name || 'Unknown Site',
                userId: userId,
                title,
                description,
                priority,
                status: 'Open',
                timestamp: serverTimestamp(),
                reporterId: userId,
                reporterRole: userRole,
            });

            setModalTitle("Success");
            setModalMessage("Issue reported successfully!");
            setIsModalOpen(true);
            // Reset form
            setTitle('');
            setDescription('');
            setSelectedSite('');
            setPriority(IssuePriorities[0]);

        } catch (error) {
            console.error("Error adding issue: ", error);
            setModalTitle("Operation Failed");
            setModalMessage("Could not report issue. Please try again.");
            setIsModalOpen(true);
        } finally {
            setLoading(false);
        }
    };

    const handleUpdateStatus = async (issueId, newStatus) => {
        if (userRole !== UserRoles.ADMIN) {
            setModalTitle("Permission Denied");
            setModalMessage("Only Administrators can update issue status.");
            setIsModalOpen(true);
            return;
        }
        try {
            const docRef = doc(db, issueCollectionPath, issueId);
            await updateDoc(docRef, { status: newStatus });
        } catch (error) {
            console.error("Error updating issue status:", error);
        }
    };

    const getPriorityStyles = (priority) => {
        switch (priority) {
            case 'Critical': return 'bg-red-600 text-white';
            case 'High': return 'bg-orange-400 text-gray-800';
            case 'Medium': return 'bg-yellow-300 text-gray-800';
            case 'Low':
            default: return 'bg-green-300 text-gray-800';
        }
    };

    return (
        <div className="p-4 md:p-8 space-y-8">
            <h2 className="text-3xl font-extrabold text-gray-800">Site Issues & Reporting</h2>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
                {/* Issue Submission Form */}
                <div className="bg-white p-6 rounded-2xl shadow-xl lg:col-span-1 h-fit">
                    <h3 className="text-xl font-bold mb-6 text-gray-800 border-b pb-3">Report New Issue</h3>
                    <form onSubmit={handleAddIssue} className="space-y-4">
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Project Site</label>
                            <select
                                value={selectedSite}
                                onChange={(e) => setSelectedSite(e.target.value)}
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                required
                            >
                                <option value="">-- Select Project Site --</option>
                                {projects.filter(p => p.status !== 'Completed').map(p => (
                                    <option key={p.id} value={p.id}>{p.name}</option>
                                ))}
                            </select>
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Title</label>
                            <input
                                type="text"
                                value={title}
                                onChange={(e) => setTitle(e.target.value)}
                                placeholder="e.g., Plywood stock missing"
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                required
                            />
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Detailed Description</label>
                            <textarea
                                value={description}
                                onChange={(e) => setDescription(e.target.value)}
                                placeholder="Describe the problem, location, and necessary action."
                                rows="3"
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                required
                            ></textarea>
                        </div>
                        <div>
                            <label className="block text-sm font-medium text-gray-700 mb-1">Priority</label>
                            <select
                                value={priority}
                                onChange={(e) => setPriority(e.target.value)}
                                className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                            >
                                {IssuePriorities.map(p => (
                                    <option key={p} value={p}>{p}</option>
                                ))}
                            </select>
                        </div>
                        <button
                            type="submit"
                            disabled={loading}
                            className="w-full p-3 mt-4 bg-red-600 text-white font-bold rounded-xl hover:bg-red-700 transition duration-300 shadow-lg disabled:bg-gray-400 flex items-center justify-center"
                        >
                            {loading ? 'Reporting...' : 'Report Issue'}
                        </button>
                    </form>
                </div>

                {/* Issue List */}
                <div className="lg:col-span-2 bg-white p-6 rounded-2xl shadow-xl">
                    <h3 className="text-xl font-bold mb-4 text-gray-800 border-b pb-3">Open Issues ({issues.filter(i => i.status === 'Open').length})</h3>
                    <div className="space-y-4 max-h-96 overflow-y-auto pr-2">
                        {issues.map(issue => (
                            <div key={issue.id} className="border p-4 rounded-xl shadow-sm hover:shadow-md transition duration-150">
                                <div className="flex justify-between items-start mb-2">
                                    <h4 className="font-bold text-gray-800 truncate pr-4">{issue.title}</h4>
                                    <span className={`px-2 py-1 text-xs font-semibold rounded-full ${getPriorityStyles(issue.priority)}`}>
                                        {issue.priority}
                                    </span>
                                </div>
                                <div className="text-sm text-gray-600 space-y-1">
                                    <p><span className="font-medium">Site:</span> {issue.siteName}</p>
                                    <p className="text-xs italic truncate"><span className="font-medium">Reported by:</span> {issue.reporterRole || 'User'} on {new Date(issue.timestamp?.seconds * 1000).toLocaleDateString()}</p>
                                    <p className="border-t pt-2 mt-2 text-gray-700">{issue.description}</p>
                                </div>
                                <div className="mt-4 flex justify-between items-center pt-3 border-t">
                                    <span className={`px-3 py-1 text-sm font-semibold rounded-full ${issue.status === 'Open' ? 'bg-yellow-100 text-yellow-800' : 'bg-green-100 text-green-800'}`}>
                                        Status: {issue.status}
                                    </span>
                                    {userRole === UserRoles.ADMIN && issue.status === 'Open' && (
                                        <button
                                            onClick={() => handleUpdateStatus(issue.id, 'Resolved')}
                                            className="text-sm bg-green-500 text-white px-3 py-1 rounded-full hover:bg-green-600 transition"
                                        >
                                            Mark Resolved
                                        </button>
                                    )}
                                </div>
                            </div>
                        ))}
                    </div>
                    {issues.length === 0 && <p className="text-center py-4 text-gray-500">No issues have been reported.</p>}
                </div>
            </div>
        </div>
    );
};

// -------------------------
// Projects (Admin/Report view) Component
// -------------------------

const Projects = ({ db, userId, projects, setIsModalOpen, setModalMessage, setModalTitle, userRole }) => {
    const [name, setName] = useState('');
    const [totalWorkUnits, setTotalWorkUnits] = useState('');
    const [status, setStatus] = useState(ProjectStatuses[0]);
    const [loading, setLoading] = useState(false);
    const isEditingAllowed = userRole === UserRoles.ADMIN;

    const projectCollectionPath = generateCollectionPath('projects', userId, true);

    const handleAddProject = async (e) => {
        e.preventDefault();
        const units = parseInt(totalWorkUnits);

        if (!name || units <= 0) {
            setModalTitle("Validation Error");
            setModalMessage("Please provide a name and total work units > 0.");
            setIsModalOpen(true);
            return;
        }

        setLoading(true);

        try {
            await addDoc(collection(db, projectCollectionPath), {
                name,
                totalWorkUnits: units,
                completedWorkUnits: 0,
                status,
                createdAt: serverTimestamp(),
            });

            setModalTitle("Success");
            setModalMessage("New project created successfully!");
            setIsModalOpen(true);
            setName('');
            setTotalWorkUnits('');
            setStatus(ProjectStatuses[0]);

        } catch (error) {
            console.error("Error adding project: ", error);
            setModalTitle("Operation Failed");
            setModalMessage("Could not create project. Please try again.");
            setIsModalOpen(true);
        } finally {
            setLoading(false);
        }
    };

    const handleUpdateStatus = async (projectId, newStatus) => {
        if (!isEditingAllowed) return;
        try {
            const docRef = doc(db, projectCollectionPath, projectId);
            await updateDoc(docRef, { status: newStatus });
        } catch (error) {
            console.error("Error updating project status:", error);
        }
    };

    const getStatusStyle = (status) => {
        switch (status) {
            case 'Active': return 'bg-green-100 text-green-800';
            case 'On Hold': return 'bg-yellow-100 text-yellow-800';
            case 'Completed': return 'bg-blue-100 text-blue-800';
            default: return 'bg-gray-100 text-gray-800';
        }
    };

    return (
        <div className="p-4 md:p-8 space-y-8">
            <h2 className="text-3xl font-extrabold text-gray-800">Project Management</h2>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
                {/* Project Creation Form (Admin Only) */}
                {isEditingAllowed && (
                    <div className="bg-white p-6 rounded-2xl shadow-xl lg:col-span-1 h-fit">
                        <h3 className="text-xl font-bold mb-6 text-gray-800 border-b pb-3">Create New Project</h3>
                        <form onSubmit={handleAddProject} className="space-y-4">
                            <div>
                                <label className="block text-sm font-medium text-gray-700 mb-1">Project Name</label>
                                <input
                                    type="text"
                                    value={name}
                                    onChange={(e) => setName(e.target.value)}
                                    placeholder="e.g., Mumbai Residential Phase 2"
                                    className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                    required
                                />
                            </div>
                            <div>
                                <label className="block text-sm font-medium text-gray-700 mb-1">Total Work Units (e.g., Sq Ft, Items)</label>
                                <input
                                    type="number"
                                    value={totalWorkUnits}
                                    onChange={(e) => setTotalWorkUnits(e.target.value)}
                                    placeholder="e.g., 5000"
                                    className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                    required
                                />
                            </div>
                            <div>
                                <label className="block text-sm font-medium text-gray-700 mb-1">Status</label>
                                <select
                                    value={status}
                                    onChange={(e) => setStatus(e.target.value)}
                                    className="w-full p-3 border border-gray-300 rounded-xl focus:ring-blue-500 focus:border-blue-500 transition"
                                >
                                    {ProjectStatuses.map(s => (
                                        <option key={s} value={s}>{s}</option>
                                    ))}
                                </select>
                            </div>
                            <button
                                type="submit"
                                disabled={loading}
                                className="w-full p-3 mt-4 bg-blue-600 text-white font-bold rounded-xl hover:bg-blue-700 transition duration-300 shadow-lg disabled:bg-gray-400 flex items-center justify-center"
                            >
                                {loading ? 'Creating...' : 'Create Project'}
                            </button>
                        </form>
                    </div>
                )}

                {/* Project List */}
                <div className={`bg-white p-6 rounded-2xl shadow-xl ${isEditingAllowed ? 'lg:col-span-2' : 'lg:col-span-3'}`}>
                    <h3 className="text-xl font-bold mb-4 text-gray-800 border-b pb-3">All Projects</h3>
                    <div className="space-y-4 max-h-96 overflow-y-auto pr-2">
                        {projects.map(project => {
                            const progress = project.completedWorkUnits && project.totalWorkUnits
                                ? Math.round((project.completedWorkUnits / project.totalWorkUnits) * 100)
                                : 0;
                            return (
                                <div key={project.id} className="border p-4 rounded-xl shadow-sm hover:shadow-md transition duration-150">
                                    <div className="flex justify-between items-start mb-2">
                                        <h4 className="font-bold text-gray-800 truncate pr-4">{project.name}</h4>
                                        <span className={`px-2 py-1 text-xs font-semibold rounded-full ${getStatusStyle(project.status)}`}>
                                            {project.status}
                                        </span>
                                    </div>
                                    <div className="text-sm text-gray-600 space-y-1">
                                        <p>
                                            <span className="font-medium">Work Progress:</span> {progress}% (
                                            {project.completedWorkUnits || 0} / {project.totalWorkUnits || 0} Units)
                                        </p>
                                        <div className="w-full bg-gray-200 rounded-full h-2.5">
                                            <div
                                                className="h-2.5 rounded-full"
                                                style={{ width: `${Math.min(100, progress)}%`, backgroundColor: progress === 100 ? '#10B981' : '#3B82F6' }}
                                            ></div>
                                        </div>
                                    </div>
                                    {isEditingAllowed && (
                                        <div className="mt-4 pt-3 border-t flex space-x-2">
                                            {ProjectStatuses.filter(s => s !== project.status).map(s => (
                                                <button
                                                    key={s}
                                                    onClick={() => handleUpdateStatus(project.id, s)}
                                                    className="text-xs bg-gray-200 text-gray-700 px-2 py-1 rounded-full hover:bg-gray-300 transition"
                                                >
                                                    Set to {s}
                                                </button>
                                            ))}
                                        </div>
                                    )}
                                </div>
                            );
                        })}
                    </div>
                    {projects.length === 0 && <p className="text-center py-4 text-gray-500">No projects defined.</p>}
                </div>
            </div>
        </div>
    );
};

// -------------------------
// Main App Component
// -------------------------

const App = () => {
    const [db, setDb] = useState(null);
    const [auth, setAuth] = useState(null);
    const [userId, setUserId] = useState(null);
    const [userRole, setUserRole] = useState(UserRoles.SUPERVISOR); // Default Role
    const [isAuthReady, setIsAuthReady] = useState(false);
    const [activeTab, setActiveTab] = useState('Dashboard');
    const [isSidebarOpen, setIsSidebarOpen] = useState(false);

    // Modal state for error/success messages
    const [isModalOpen, setIsModalOpen] = useState(false);
    const [modalTitle, setModalTitle] = useState('');
    const [modalMessage, setModalMessage] = useState('');

    // Shared Data State
    const [projects, setProjects] = useState([]);
    const [expenses, setExpenses] = useState([]);
    const [attendance, setAttendance] = useState([]);

    // --- FIREBASE INITIALIZATION AND AUTHENTICATION ---
    useEffect(() => {
        try {
            // Set logging level to Debug for visibility in development
            setLogLevel('debug');
            const app = initializeApp(firebaseConfig);
            const firestore = getFirestore(app);
            const authInstance = getAuth(app);
            setDb(firestore);
            setAuth(authInstance);

            // Authentication Logic: Use anonymous sign-in for deployed version
            // This replaces the complex logic that depended on the special __initial_auth_token
            signInAnonymously(authInstance)
                .then((userCredential) => {
                    setUserId(userCredential.user.uid);
                    // Determine role (mocked for simplicity; in reality, this would be a DB lookup)
                    setUserRole(userCredential.user.uid.includes('admin') ? UserRoles.ADMIN : UserRoles.SUPERVISOR);
                    setIsAuthReady(true);
                })
                .catch(error => {
                    console.error("Anonymous Sign-In Failed:", error);
                    setIsAuthReady(true); // Still set ready, even if auth failed
                });
            
        } catch (e) {
            console.error("Firebase initialization error:", e);
            setModalTitle("Configuration Error");
            setModalMessage("Firebase failed to initialize. Check your configuration keys.");
            setIsModalOpen(true);
        }
    }, []);

    // --- DATA SUBSCRIPTIONS (onSnapshot) ---
    useEffect(() => {
        if (!db || !isAuthReady || !userId) return;

        // 1. Projects (Public/Shared Data)
        const projectsPath = generateCollectionPath('projects', userId, true);
        const projectsQuery = query(collection(db, projectsPath));
        const unsubscribeProjects = onSnapshot(projectsQuery, (snapshot) => {
            const list = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setProjects(list.sort((a, b) => a.name.localeCompare(b.name)));
        }, (error) => console.error("Error fetching projects:", error));

        // 2. Expenses (Public/Shared Data)
        const expensesPath = generateCollectionPath('expenses', userId, true);
        const expensesQuery = query(collection(db, expensesPath));
        const unsubscribeExpenses = onSnapshot(expensesQuery, (snapshot) => {
            const list = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setExpenses(list);
        }, (error) => console.error("Error fetching expenses:", error));
        
        // 3. Attendance (Public/Shared Data)
        const attendancePath = generateCollectionPath('attendance', userId, true);
        const attendanceQuery = query(collection(db, attendancePath));
        const unsubscribeAttendance = onSnapshot(attendanceQuery, (snapshot) => {
            const list = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setAttendance(list);
        }, (error) => console.error("Error fetching attendance:", error));

        return () => {
            unsubscribeProjects();
            unsubscribeExpenses();
            unsubscribeAttendance();
        };
    }, [db, isAuthReady, userId]);

    const handleSignOut = useCallback(() => {
        if (auth) {
            signOut(auth)
                .then(() => {
                    window.location.reload(); // Simple refresh to clear state
                })
                .catch((error) => {
                    console.error("Sign Out Error:", error);
                });
        }
    }, [auth]);

    const renderContent = () => {
        if (!isAuthReady || !db || !userId) {
            return (
                <div className="flex justify-center items-center h-full text-gray-500">
                    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500 mr-3"></div>
                    Loading System Resources...
                </div>
            );
        }

        const commonProps = {
            db, userId, projects, setIsModalOpen, setModalMessage, setModalTitle, userRole
        };

        switch (activeTab) {
            case 'Dashboard':
                return <Dashboard {...commonProps} expenses={expenses} attendance={attendance} />;
            case 'Attendance':
                return <Attendance {...commonProps} />;
            case 'Work Progress':
                // Using Projects component for now as a general site management area
                return <Projects {...commonProps} />;
            case 'Issues':
                return <Issues {...commonProps} />;
            case 'Expenses':
                return <Expenses {...commonProps} />;
            case 'Reports':
                // Using Projects component for now as a general site management area
                return <Projects {...commonProps} />; 
            default:
                return <Dashboard {...commonProps} expenses={expenses} attendance={attendance} />;
        }
    };

    const navItems = [
        { title: 'Dashboard', icon: Home, tab: 'Dashboard' },
        { title: 'Attendance', icon: Clock, tab: 'Attendance' },
        { title: 'Work Progress', icon: Layers, tab: 'Work Progress' },
        { title: 'Issues & Safety', icon: AlertTriangle, tab: 'Issues' },
        { title: 'Expenses & Claims', icon: DollarSign, tab: 'Expenses' },
        { title: 'Projects List', icon: Folder, tab: 'Reports' },
    ];

    return (
        <div className="min-h-screen flex bg-gray-50 font-sans">
            <Modal
                title={modalTitle}
                message={modalMessage}
                isOpen={isModalOpen}
                onClose={() => setIsModalOpen(false)}
            />

            {/* Sidebar (Desktop) */}
            <div className={`w-64 flex-shrink-0 bg-white shadow-2xl hidden md:flex flex-col border-r transition-all duration-300`}>
                <div className="p-6 text-center border-b">
                    <h1 className="text-2xl font-black text-blue-600">ALSORG FOMS</h1>
                    <p className="text-xs text-gray-500 mt-1">{userRole}</p>
                </div>
                <div className="flex-grow p-4 space-y-2">
                    {navItems.map(item => (
                        <NavItem
                            key={item.tab}
                            title={item.title}
                            icon={item.icon}
                            active={activeTab === item.tab}
                            onClick={() => setActiveTab(item.tab)}
                        />
                    ))}
                </div>
                <NavFooter userId={userId} onSignOut={handleSignOut} />
            </div>

            {/* Mobile Sidebar Overlay */}
            {isSidebarOpen && (
                <div
                    className="fixed inset-0 bg-black bg-opacity-50 z-40 md:hidden"
                    onClick={() => setIsSidebarOpen(false)}
                ></div>
            )}

            {/* Mobile Sidebar */}
            <div className={`fixed inset-y-0 left-0 w-64 bg-white shadow-2xl z-50 transform ${isSidebarOpen ? 'translate-x-0' : '-translate-x-full'} transition-transform duration-300 md:hidden flex flex-col`}>
                <div className="p-6 flex justify-between items-center border-b">
                    <h1 className="text-2xl font-black text-blue-600">FOMS</h1>
                    <button onClick={() => setIsSidebarOpen(false)} className="text-gray-600 hover:text-gray-900">
                        <X className="w-6 h-6" />
                    </button>
                </div>
                <div className="flex-grow p-4 space-y-2">
                    {navItems.map(item => (
                        <NavItem
                            key={item.tab}
                            title={item.title}
                            icon={item.icon}
                            active={activeTab === item.tab}
                            onClick={() => {
                                setActiveTab(item.tab);
                                setIsSidebarOpen(false);
                            }}
                        />
                    ))}
                </div>
                <NavFooter userId={userId} onSignOut={handleSignOut} />
            </div>

            {/* Main Content Area */}
            <div className="flex-grow flex flex-col overflow-y-auto">
                {/* Mobile Header */}
                <header className="md:hidden sticky top-0 bg-white shadow-md p-4 flex justify-between items-center z-30">
                    <button onClick={() => setIsSidebarOpen(true)} className="text-gray-700 hover:text-gray-900">
                        <MenuIcon className="w-6 h-6" />
                    </button>
                    <h2 className="text-lg font-semibold text-gray-800">{activeTab}</h2>
                    <LogOut className="w-6 h-6 opacity-0" /> {/* Placeholder for alignment */}
                </header>

                <main className="flex-grow pb-8">
                    {renderContent()}
                </main>
            </div>
        </div>
    );
};

export default App;
