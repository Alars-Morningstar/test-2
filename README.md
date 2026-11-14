import React, { useState, useMemo, useEffect, useRef } from 'react';
import {
  PhoneCall, CheckCircle2, XCircle, Clock, Users, Calendar, Plus,
  Search, Download, Upload, RotateCcw, TrendingUp, UserPlus, Award,
  AlertCircle, Check, X, Headphones, CalendarDays, BarChart3, UserCheck,
  Trash2, Shield, Lock, Unlock, LogOut, Eye, EyeOff, KeyRound, Activity,
  ArrowUpDown, Target
} from 'lucide-react';

// --- VERSIONED STORAGE KEYS (v4) TO APPLY HARDCODED ADMIN CREDENTIALS IMMEDIATELY ---
const STORAGE_KEYS = {
  WEEKS: 'dialify_crm_v4_weeks',
  AGENTS: 'dialify_crm_v4_agents',
  SELECTED_WEEK: 'dialify_crm_v4_selected_week',
  WEEKLY_GOAL: 'dialify_crm_v4_goal',
  ADMINS: 'dialify_crm_v4_admins',
  AUTH_SESSION: 'dialify_crm_v4_session',
  AUDIT_LOG: 'dialify_crm_v4_audit'
};

// --- 5-DAY WORKWEEK DEFINITIONS ---
const DAYS = [
  { key: 'd1', label: 'Day 1', subLabel: 'Mon / Tue', short: 'Day 1' },
  { key: 'd2', label: 'Day 2', subLabel: 'Tue / Wed', short: 'Day 2' },
  { key: 'd3', label: 'Day 3', subLabel: 'Wed / Thu', short: 'Day 3' },
  { key: 'd4', label: 'Day 4', subLabel: 'Thu / Fri', short: 'Day 4' },
  { key: 'd5', label: 'Day 5', subLabel: 'Fri / Sat', short: 'Day 5' }
];

const ATTENDANCE_STATUSES = ['Present', 'Late', 'Half-Day', 'Absent'];

const CAMPAIGN_CONFIG = {
  'Timeshare': {
    shift: 'Mon–Fri (9:00 PM - 7:00 AM)',
    days: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri']
  },
  'Travel Club': {
    shift: 'Tue–Sat (11:55 PM - 9:00 AM)',
    days: ['Tue', 'Wed', 'Thu', 'Fri', 'Sat']
  },
  'Home Improvement': {
    shift: 'Mon–Fri (11:55 PM - 9:00 AM)',
    days: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri']
  }
};

const CAMPAIGNS = ['All Campaigns', 'Timeshare', 'Travel Club', 'Home Improvement'];
const SHIFTS = [
  'Mon–Fri (9:00 PM - 7:00 AM)',
  'Tue–Sat (11:55 PM - 9:00 AM)',
  'Mon–Fri (11:55 PM - 9:00 AM)'
];

// --- INITIAL 5-DAY WEEKS DATA FOR DIALIFY ASIA ---
const INITIAL_WEEKS = [
  {
    id: 'wk-nov-9',
    label: 'Week of Nov 9 – Nov 13, 2026 (Current)',
    dateRange: 'Nov 9 - Nov 13, 2026'
  },
  {
    id: 'wk-nov-2',
    label: 'Week of Nov 2 – Nov 6, 2026',
    dateRange: 'Nov 2 - Nov 6, 2026'
  },
  {
    id: 'wk-oct-26',
    label: 'Week of Oct 26 – Oct 30, 2026',
    dateRange: 'Oct 26 - Oct 30, 2026'
  }
];

// --- HARDCODED AUTHORIZED ADMIN ACCOUNTS ---
const DEFAULT_ADMINS = [
  {
    id: 'adm-1',
    username: 'admin1',
    passcode: 'Boom@428216',
    name: 'Admin 1',
    role: 'Super Admin'
  },
  {
    id: 'adm-2',
    username: 'alars',
    passcode: 'Boom@7243',
    name: 'Alars',
    role: 'Super Admin'
  }
];

const INITIAL_AUDIT_LOG = [
  {
    id: 'log-init-1',
    timestamp: 'Nov 14, 2026 • 06:00 PM',
    actor: 'System Security',
    action: 'Initialized Dialify Asia 5-Day Weekly Floor CRM v4 security gate'
  }
];

const INITIAL_AGENTS = [
  {
    id: 'ag-1',
    name: 'Tanvir Ahmed',
    agentCode: 'DA-101',
    campaign: 'Timeshare',
    shift: 'Mon–Fri (9:00 PM - 7:00 AM)',
    phone: '01711-234567',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 4, shows: 3 },
          d2: { booked: 5, shows: 4 },
          d3: { booked: 3, shows: 3 },
          d4: { booked: 6, shows: 5 },
          d5: { booked: 4, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Top floor closer this week'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 4, shows: 3 },
          d4: { booked: 5, shows: 4 },
          d5: { booked: 3, shows: 2 }
        },
        att: { d1: 'Present', d2: 'Late', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Strong consistency'
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 4, shows: 3 },
          d2: { booked: 3, shows: 2 },
          d3: { booked: 5, shows: 4 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 4, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: '100% attendance'
      }
    }
  },
  {
    id: 'ag-2',
    name: 'Nusrat Jahan',
    agentCode: 'DA-102',
    campaign: 'Travel Club',
    shift: 'Tue–Sat (11:55 PM - 9:00 AM)',
    phone: '01819-876543',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 4, shows: 3 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 5, shows: 4 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 4, shows: 4 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: '100% on-time login all 5 days'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 3, shows: 3 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 3, shows: 2 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 4, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Zero absences'
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 4, shows: 3 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 3, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Half-Day', d4: 'Present', d5: 'Present' },
        note: 'Early leave Wed'
      }
    }
  },
  {
    id: 'ag-3',
    name: 'Rafiqul Islam',
    agentCode: 'DA-103',
    campaign: 'Home Improvement',
    shift: 'Mon–Fri (11:55 PM - 9:00 AM)',
    phone: '01912-345678',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 4, shows: 2 },
          d3: { booked: 3, shows: 2 },
          d4: { booked: 2, shows: 1 },
          d5: { booked: 4, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Late', d3: 'Present', d4: 'Late', d5: 'Present' },
        note: 'Traffic delay at Mirpur 10 on Tue & Thu'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 2, shows: 1 },
          d3: { booked: 4, shows: 2 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 3, shows: 2 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Absent', d4: 'Present', d5: 'Present' },
        note: 'Sick leave on Day 3'
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 2, shows: 1 },
          d2: { booked: 3, shows: 2 },
          d3: { booked: 3, shows: 2 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 2, shows: 1 }
        },
        att: { d1: 'Present', d2: 'Late', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: ''
      }
    }
  },
  {
    id: 'ag-4',
    name: 'Sadia Afrin',
    agentCode: 'DA-104',
    campaign: 'Timeshare',
    shift: 'Mon–Fri (9:00 PM - 7:00 AM)',
    phone: '01678-901234',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 5, shows: 4 },
          d2: { booked: 5, shows: 4 },
          d3: { booked: 4, shows: 3 },
          d4: { booked: 5, shows: 4 },
          d5: { booked: 6, shows: 5 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Highest show rate on Timeshare campaign'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 4, shows: 3 },
          d2: { booked: 5, shows: 4 },
          d3: { booked: 4, shows: 4 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 5, shows: 4 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Star performer'
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 4, shows: 3 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 5, shows: 4 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 4, shows: 4 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: ''
      }
    }
  },
  {
    id: 'ag-5',
    name: 'Mehedi Hasan',
    agentCode: 'DA-105',
    campaign: 'Travel Club',
    shift: 'Tue–Sat (11:55 PM - 9:00 AM)',
    phone: '01755-667788',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 3, shows: 1 },
          d2: { booked: 2, shows: 1 },
          d3: { booked: 3, shows: 1 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 2, shows: 1 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Half-Day', d4: 'Present', d5: 'Present' },
        note: 'Needs coaching on prospect confirmation'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 2, shows: 1 },
          d2: { booked: 3, shows: 1 },
          d3: { booked: 2, shows: 1 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 2, shows: 1 }
        },
        att: { d1: 'Late', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Absent' },
        note: 'Uninformed absence on Friday'
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 2, shows: 1 },
          d3: { booked: 3, shows: 1 },
          d4: { booked: 2, shows: 1 },
          d5: { booked: 3, shows: 2 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: ''
      }
    }
  },
  {
    id: 'ag-6',
    name: 'Farhana Yasmin',
    agentCode: 'DA-106',
    campaign: 'Home Improvement',
    shift: 'Mon–Fri (11:55 PM - 9:00 AM)',
    phone: '01534-223344',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 4, shows: 3 },
          d2: { booked: 3, shows: 2 },
          d3: { booked: 0, shows: 0 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 3, shows: 2 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Absent', d4: 'Present', d5: 'Present' },
        note: 'Medical leave on Wednesday (Day 3)'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 3, shows: 2 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 4, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Good week'
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 3, shows: 2 },
          d3: { booked: 4, shows: 3 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 3, shows: 2 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: ''
      }
    }
  },
  {
    id: 'ag-7',
    name: 'Kamrul Hasan',
    agentCode: 'DA-107',
    campaign: 'Timeshare',
    shift: 'Mon–Fri (9:00 PM - 7:00 AM)',
    phone: '01311-998877',
    weeklyData: {
      'wk-nov-9': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 4, shows: 3 },
          d3: { booked: 4, shows: 3 },
          d4: { booked: 3, shows: 3 },
          d5: { booked: 4, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: 'Strong rebuttal handling all week'
      },
      'wk-nov-2': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 3, shows: 2 },
          d3: { booked: 4, shows: 3 },
          d4: { booked: 3, shows: 2 },
          d5: { booked: 3, shows: 3 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Late', d5: 'Present' },
        note: ''
      },
      'wk-oct-26': {
        perf: {
          d1: { booked: 3, shows: 2 },
          d2: { booked: 3, shows: 2 },
          d3: { booked: 3, shows: 2 },
          d4: { booked: 4, shows: 3 },
          d5: { booked: 3, shows: 2 }
        },
        att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
        note: ''
      }
    }
  }
];

// Helper to create blank 5-day structure for a week
const createEmptyWeekRecord = () => ({
  perf: {
    d1: { booked: 0, shows: 0 },
    d2: { booked: 0, shows: 0 },
    d3: { booked: 0, shows: 0 },
    d4: { booked: 0, shows: 0 },
    d5: { booked: 0, shows: 0 }
  },
  weekShows: 0,
  att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' },
  note: ''
});

// Strict schema-validated localStorage loaders
const loadValidatedArray = (key, fallback, requiredField) => {
  try {
    const raw = localStorage.getItem(key);
    if (!raw) return fallback;
    const parsed = JSON.parse(raw);
    if (!Array.isArray(parsed) || parsed.length === 0) return fallback;
    if (requiredField && (!parsed[0] || typeof parsed[0] !== 'object' || !(requiredField in parsed[0]))) {
      return fallback;
    }
    return parsed;
  } catch (e) {
    return fallback;
  }
};

const loadValidatedPrimitive = (key, fallback, expectedType) => {
  try {
    const raw = localStorage.getItem(key);
    if (!raw) return fallback;
    const parsed = JSON.parse(raw);
    if (typeof parsed !== expectedType) return fallback;
    return parsed;
  } catch (e) {
    return fallback;
  }
};

export default function App() {
  // --- ADMIN AUTHENTICATION & SECURITY STATES ---
  const [adminAccounts, setAdminAccounts] = useState(() =>
    loadValidatedArray(STORAGE_KEYS.ADMINS, DEFAULT_ADMINS, 'username')
  );
  const [auditLogs, setAuditLogs] = useState(() =>
    loadValidatedArray(STORAGE_KEYS.AUDIT_LOG, INITIAL_AUDIT_LOG, 'action')
  );
  // Default to unauthenticated on initial load unless a valid session object is in sessionStorage
  const [currentUser, setCurrentUser] = useState(() => {
    try {
      const savedSession =
        sessionStorage.getItem(STORAGE_KEYS.AUTH_SESSION) ||
        localStorage.getItem(STORAGE_KEYS.AUTH_SESSION);
      if (savedSession) {
        const parsed = JSON.parse(savedSession);
        if (parsed && typeof parsed === 'object' && parsed.username && parsed.role) {
          return parsed;
        }
      }
      return null;
    } catch (e) {
      return null;
    }
  });
  const [isScreenLocked, setIsScreenLocked] = useState(false);

  // Login Form States
  const [loginUsername, setLoginUsername] = useState('');
  const [loginPasscode, setLoginPasscode] = useState('');
  const [showPasscode, setShowPasscode] = useState(false);
  const [rememberSession, setRememberSession] = useState(true);
  const [loginError, setLoginError] = useState('');
  const [failedAttempts, setFailedAttempts] = useState(0);
  const [lockoutSeconds, setLockoutSeconds] = useState(0);

  // Admin Security Modal States
  const [showSecurityModal, setShowSecurityModal] = useState(false);
  const [newAdminForm, setNewAdminForm] = useState({
    username: '',
    name: '',
    role: 'Super Admin',
    passcode: ''
  });
  const [passwordChangeForm, setPasswordChangeForm] = useState({
    targetUsername: 'admin1',
    newPasscode: ''
  });

  // Navigation & Persistent States
  const [activeTab, setActiveTab] = useState('performance'); // 'performance' | 'attendance' | 'analytics' | 'agents'
  const [weeks, setWeeks] = useState(() =>
    loadValidatedArray(STORAGE_KEYS.WEEKS, INITIAL_WEEKS, 'dateRange')
  );
  const [selectedWeekId, setSelectedWeekId] = useState(() =>
    loadValidatedPrimitive(STORAGE_KEYS.SELECTED_WEEK, INITIAL_WEEKS[0].id, 'string')
  );
  const [agents, setAgents] = useState(() =>
    loadValidatedArray(STORAGE_KEYS.AGENTS, INITIAL_AGENTS, 'agentCode')
  );
  const [weeklyBookingGoal, setWeeklyBookingGoal] = useState(() =>
    loadValidatedPrimitive(STORAGE_KEYS.WEEKLY_GOAL, 20, 'number')
  );

  // Filter & Sorting States
  const [searchQuery, setSearchQuery] = useState('');
  const [campaignFilter, setCampaignFilter] = useState('All Campaigns');
  const [sortBy, setSortBy] = useState('default');
  const [toast, setToast] = useState(null);
  const [lastSavedTime, setLastSavedTime] = useState('Just now');
  const [agentToDelete, setAgentToDelete] = useState(null);
  const [confirmResetModal, setConfirmResetModal] = useState(false);

  // Modals
  const [showAgentModal, setShowAgentModal] = useState(false);
  const [showNewWeekModal, setShowNewWeekModal] = useState(false);
  const [showQuickLogModal, setShowQuickLogModal] = useState(false);
  const [selectedAgentProfile, setSelectedAgentProfile] = useState(null);

  const fileInputRef = useRef(null);

  // Form States
  const [newAgent, setNewAgent] = useState({
    name: '',
    agentCode: 'DA-108',
    campaign: 'Timeshare',
    shift: 'Mon–Fri (9:00 PM - 7:00 AM)',
    phone: ''
  });

  const [newWeekLabel, setNewWeekLabel] = useState('');

  const [quickLog, setQuickLog] = useState({
    agentId: INITIAL_AGENTS[0].id,
    dayKey: 'd1',
    addBooked: 1,
    addWeekShows: 0
  });

  const notify = (msg) => setToast(msg);

  useEffect(() => {
    if (!toast) return;
    const timer = setTimeout(() => setToast(null), 3200);
    return () => clearTimeout(timer);
  }, [toast]);

  // --- BRUTE-FORCE LOCKOUT COUNTDOWN TIMER ---
  useEffect(() => {
    if (lockoutSeconds <= 0) return;
    const timer = setInterval(() => {
      setLockoutSeconds(prev => {
        if (prev <= 1) {
          setFailedAttempts(0);
          setLoginError('');
          return 0;
        }
        return prev - 1;
      });
    }, 1000);
    return () => clearInterval(timer);
  }, [lockoutSeconds]);

  // --- AUTO-SAVE TO LOCALSTORAGE ---
  useEffect(() => {
    try {
      localStorage.setItem(STORAGE_KEYS.WEEKS, JSON.stringify(weeks));
      localStorage.setItem(STORAGE_KEYS.AGENTS, JSON.stringify(agents));
      localStorage.setItem(STORAGE_KEYS.SELECTED_WEEK, JSON.stringify(selectedWeekId));
      localStorage.setItem(STORAGE_KEYS.WEEKLY_GOAL, JSON.stringify(weeklyBookingGoal));
      localStorage.setItem(STORAGE_KEYS.ADMINS, JSON.stringify(adminAccounts));
      localStorage.setItem(STORAGE_KEYS.AUDIT_LOG, JSON.stringify(auditLogs));
      const now = new Date();
      setLastSavedTime(
        now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', second: '2-digit' })
      );
    } catch (e) {
      // Safe fallback in restricted browser storage environments
    }
  }, [weeks, agents, selectedWeekId, weeklyBookingGoal, adminAccounts, auditLogs]);

  // Helper to record security & floor audit events
  const appendAuditLog = (actionText, actorOverride = null) => {
    const now = new Date();
    const formatted = `${now.toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' })} • ${now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`;
    const entry = {
      id: `log-${Date.now()}-${Math.random().toString(36).slice(2, 6)}`,
      timestamp: formatted,
      actor: actorOverride || currentUser?.name || 'Admin',
      action: actionText
    };
    setAuditLogs(prev => [entry, ...(Array.isArray(prev) ? prev.slice(0, 49) : [])]);
  };

  // --- ADMIN AUTHENTICATION HANDLERS ---
  const executeLoginWithCredentials = (usernameInput, passcodeInput) => {
    if (lockoutSeconds > 0) return;

    const cleanUser = String(usernameInput || '').trim().toLowerCase();
    const matched = adminAccounts.find(
      a => a.username.toLowerCase() === cleanUser && a.passcode === passcodeInput
    );

    if (!matched) {
      const nextAttempts = failedAttempts + 1;
      setFailedAttempts(nextAttempts);
      if (nextAttempts >= 5) {
        setLockoutSeconds(30);
        setLoginError('Too many failed attempts. Terminal locked for 30 seconds.');
        appendAuditLog(`Security Alert: 5 failed login attempts for "${cleanUser}"`, 'Security Gate');
      } else {
        setLoginError(`Invalid Admin credentials (${5 - nextAttempts} attempts remaining).`);
      }
      return;
    }

    const sessionUser = {
      id: matched.id,
      username: matched.username,
      name: matched.name,
      role: matched.role,
      loginTime: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
    };

    setCurrentUser(sessionUser);
    setIsScreenLocked(false);
    setFailedAttempts(0);
    setLoginError('');
    setLoginPasscode('');

    try {
      sessionStorage.setItem(STORAGE_KEYS.AUTH_SESSION, JSON.stringify(sessionUser));
      if (rememberSession) {
        localStorage.setItem(STORAGE_KEYS.AUTH_SESSION, JSON.stringify(sessionUser));
      }
    } catch (err) {
      // Ignore storage errors
    }

    appendAuditLog(`Signed in to Dialify Asia CRM (${matched.role})`, matched.name);
    notify(`Welcome back, ${matched.name}!`);
  };

  const handleAdminLogin = (e) => {
    e.preventDefault();
    executeLoginWithCredentials(loginUsername, loginPasscode);
  };

  const handleEmergencyClearAndReset = () => {
    try {
      Object.values(STORAGE_KEYS).forEach(k => {
        localStorage.removeItem(k);
        sessionStorage.removeItem(k);
      });
    } catch (e) {
      // Ignore
    }
    setAdminAccounts(DEFAULT_ADMINS);
    setAuditLogs(INITIAL_AUDIT_LOG);
    setWeeks(INITIAL_WEEKS);
    setAgents(INITIAL_AGENTS);
    setSelectedWeekId(INITIAL_WEEKS[0].id);
    setWeeklyBookingGoal(20);
    setCurrentUser(null);
    setIsScreenLocked(false);
    setFailedAttempts(0);
    setLockoutSeconds(0);
    setLoginUsername('');
    setLoginPasscode('');
    setLoginError('');
    notify('Cleared cached storage & restored factory settings!');
  };

  const handleLockScreen = () => {
    setIsScreenLocked(true);
    setLoginUsername(currentUser?.username || '');
    setLoginPasscode('');
    setLoginError('');
    appendAuditLog('Locked CRM terminal screen');
  };

  const handleSignOut = () => {
    appendAuditLog('Signed out of Dialify Asia CRM');
    setCurrentUser(null);
    setIsScreenLocked(false);
    setLoginUsername('');
    setLoginPasscode('');
    setLoginError('');
    try {
      localStorage.removeItem(STORAGE_KEYS.AUTH_SESSION);
      sessionStorage.removeItem(STORAGE_KEYS.AUTH_SESSION);
    } catch (err) {
      // Ignore
    }
  };

  const handleChangeAdminPasscode = (e) => {
    e.preventDefault();
    if (!passwordChangeForm.newPasscode || passwordChangeForm.newPasscode.length < 4) {
      notify('New passcode must be at least 4 characters.');
      return;
    }
    setAdminAccounts(prev =>
      prev.map(acc =>
        acc.username === passwordChangeForm.targetUsername
          ? { ...acc, passcode: passwordChangeForm.newPasscode }
          : acc
      )
    );
    appendAuditLog(`Updated security passcode for @${passwordChangeForm.targetUsername}`);
    setPasswordChangeForm(prev => ({ ...prev, newPasscode: '' }));
    notify(`Updated passcode for @${passwordChangeForm.targetUsername}!`);
  };

  const handleCreateNewAdminAccount = (e) => {
    e.preventDefault();
    const cleanUsername = newAdminForm.username.trim().toLowerCase();
    if (!cleanUsername || !newAdminForm.name.trim() || !newAdminForm.passcode.trim()) return;
    if (adminAccounts.some(a => a.username.toLowerCase() === cleanUsername)) {
      notify('Admin username already exists!');
      return;
    }

    const created = {
      id: `adm-${Date.now()}`,
      username: cleanUsername,
      name: newAdminForm.name.trim(),
      role: newAdminForm.role,
      passcode: newAdminForm.passcode.trim()
    };

    setAdminAccounts(prev => [...prev, created]);
    appendAuditLog(`Created new admin account: @${created.username} (${created.role})`);
    setNewAdminForm({ username: '', name: '', role: 'Super Admin', passcode: '' });
    notify(`Added admin @${created.username} (${created.role})`);
  };

  const handleRemoveAdminAccount = (adm) => {
    if (adm.username === 'admin1' || adminAccounts.length <= 1) {
      notify('Primary Super Admin account cannot be removed.');
      return;
    }
    setAdminAccounts(prev => prev.filter(a => a.id !== adm.id));
    appendAuditLog(`Removed admin account: @${adm.username}`);
    notify(`Removed admin @${adm.username}`);
  };

  const isSuperAdmin = currentUser?.role === 'Super Admin';

  const currentWeekObj = useMemo(() => {
    const safeWeeks = Array.isArray(weeks) && weeks.length > 0 ? weeks : INITIAL_WEEKS;
    return safeWeeks.find(w => w.id === selectedWeekId) || safeWeeks[0];
  }, [weeks, selectedWeekId]);

  // --- HELPER CALCULATIONS ---
  const getShowRate = (shows, booked) => {
    if (!booked || booked <= 0) return 0;
    return Math.round((shows / booked) * 100);
  };

  const getShowRateBadge = (rate) => {
    if (rate >= 70) {
      return {
        bg: 'bg-[#ff5a00]/20 text-[#ff5a00] border-[#ff5a00]/50',
        bar: 'bg-[#ff5a00]',
        tag: 'Excellent'
      };
    }
    if (rate >= 50) {
      return {
        bg: 'bg-amber-500/20 text-amber-400 border-amber-500/40',
        bar: 'bg-amber-500',
        tag: 'Average'
      };
    }
    return {
      bg: 'bg-rose-500/20 text-rose-400 border-rose-500/40',
      bar: 'bg-rose-500',
      tag: 'Low'
    };
  };

  const getAttendanceStyle = (status) => {
    switch (status) {
      case 'Present':
        return 'bg-[#ff5a00]/20 text-[#ff5a00] border-[#ff5a00]/50 hover:bg-[#ff5a00]/30';
      case 'Late':
        return 'bg-amber-500/20 text-amber-300 border-amber-500/40 hover:bg-amber-500/30';
      case 'Half-Day':
        return 'bg-sky-500/20 text-sky-300 border-sky-500/40 hover:bg-sky-500/30';
      case 'Absent':
        return 'bg-rose-500/20 text-rose-400 border-rose-500/40 hover:bg-rose-500/30';
      default:
        return 'bg-[#1e2128] text-slate-300 border-[#454d5e]';
    }
  };

  // Compute an agent's 5-day performance totals for the selected week
  const getAgentWeekPerf = (agent, weekId = selectedWeekId) => {
    const record = agent?.weeklyData?.[weekId] || createEmptyWeekRecord();
    let totalBooked = 0;
    let fallbackShows = 0;

    DAYS.forEach(d => {
      const dayObj = record?.perf?.[d.key] || { booked: 0, shows: 0 };
      totalBooked += Number(dayObj.booked) || 0;
      fallbackShows += Number(dayObj.shows) || 0;
    });

    const rawShows = typeof record.weekShows === 'number' ? record.weekShows : fallbackShows;
    const totalShows = Math.min(totalBooked, Math.max(0, rawShows));
    const noShows = Math.max(0, totalBooked - totalShows);
    const showRate = getShowRate(totalShows, totalBooked);

    return {
      days: record?.perf || createEmptyWeekRecord().perf,
      totalBooked,
      totalShows,
      noShows,
      showRate
    };
  };

  // Compute an agent's 5-day attendance score for the selected week
  const getAgentWeekAtt = (agent, weekId = selectedWeekId) => {
    const record = agent?.weeklyData?.[weekId] || createEmptyWeekRecord();
    let scoreDays = 0;
    let presentCount = 0;
    let lateCount = 0;
    let halfDayCount = 0;
    let absentCount = 0;

    DAYS.forEach(d => {
      const st = record?.att?.[d.key] || 'Present';
      if (st === 'Present') {
        scoreDays += 1;
        presentCount += 1;
      } else if (st === 'Late') {
        scoreDays += 1;
        lateCount += 1;
      } else if (st === 'Half-Day') {
        scoreDays += 0.5;
        halfDayCount += 1;
      } else if (st === 'Absent') {
        absentCount += 1;
      }
    });

    const attPercent = Math.round((scoreDays / 5) * 100);

    return {
      days: record?.att || createEmptyWeekRecord().att,
      note: record?.note || '',
      scoreDays,
      presentCount,
      lateCount,
      halfDayCount,
      absentCount,
      attPercent
    };
  };

  // --- FILTERED & SORTED AGENTS ---
  const filteredAgents = useMemo(() => {
    const safeAgents = Array.isArray(agents) ? agents : INITIAL_AGENTS;
    const list = safeAgents.filter(a => {
      if (!a || typeof a !== 'object') return false;
      const matchesSearch =
        String(a.name || '').toLowerCase().includes(searchQuery.toLowerCase()) ||
        String(a.agentCode || '').toLowerCase().includes(searchQuery.toLowerCase()) ||
        String(a.campaign || '').toLowerCase().includes(searchQuery.toLowerCase());
      const matchesCamp = campaignFilter === 'All Campaigns' || a.campaign === campaignFilter;
      return matchesSearch && matchesCamp;
    });

    if (sortBy === 'default') return list;

    return [...list].sort((a, b) => {
      const perfA = getAgentWeekPerf(a, selectedWeekId);
      const perfB = getAgentWeekPerf(b, selectedWeekId);

      if (sortBy === 'booked-desc') return perfB.totalBooked - perfA.totalBooked;
      if (sortBy === 'shows-desc') return perfB.totalShows - perfA.totalShows;
      if (sortBy === 'rate-desc') return perfB.showRate - perfA.showRate;
      if (sortBy === 'name-asc') return String(a.name).localeCompare(String(b.name));
      return 0;
    });
  }, [agents, searchQuery, campaignFilter, sortBy, selectedWeekId]);

  // Rank map based on total shows (tie-breaker: showRate)
  const agentRankMap = useMemo(() => {
    const ranked = [...filteredAgents].sort((a, b) => {
      const pA = getAgentWeekPerf(a, selectedWeekId);
      const pB = getAgentWeekPerf(b, selectedWeekId);
      if (pB.totalShows !== pA.totalShows) return pB.totalShows - pA.totalShows;
      return pB.showRate - pA.showRate;
    });
    const map = {};
    ranked.forEach((ag, idx) => {
      const p = getAgentWeekPerf(ag, selectedWeekId);
      if (p.totalShows > 0) {
        map[ag.id] = idx + 1;
      }
    });
    return map;
  }, [filteredAgents, selectedWeekId, agents]);

  // --- WEEKLY SUMMARY KPIs (FOR SELECTED 5-DAY WEEK) ---
  const weeklySummary = useMemo(() => {
    let weekBooked = 0;
    let weekShows = 0;
    let bestAgent = null;
    let bestShows = -1;

    const dayTotals = {
      d1: { booked: 0, shows: 0 },
      d2: { booked: 0, shows: 0 },
      d3: { booked: 0, shows: 0 },
      d4: { booked: 0, shows: 0 },
      d5: { booked: 0, shows: 0 }
    };

    let totalPossibleDays = filteredAgents.length * 5;
    let totalAttendedDays = 0;
    let totalLateSlots = 0;
    let totalAbsentSlots = 0;
    let totalPresentOnTimeSlots = 0;

    filteredAgents.forEach(ag => {
      const perf = getAgentWeekPerf(ag, selectedWeekId);
      const att = getAgentWeekAtt(ag, selectedWeekId);

      weekBooked += perf.totalBooked;
      weekShows += perf.totalShows;

      if (perf.totalShows > bestShows) {
        bestShows = perf.totalShows;
        bestAgent = {
          ...ag,
          weekShows: perf.totalShows,
          weekBooked: perf.totalBooked,
          showRate: perf.showRate
        };
      }

      DAYS.forEach(d => {
        dayTotals[d.key].booked += Number(perf.days?.[d.key]?.booked) || 0;
      });

      totalAttendedDays += att.scoreDays;
      totalLateSlots += att.lateCount;
      totalAbsentSlots += att.absentCount;
      totalPresentOnTimeSlots += att.presentCount;
    });

    const weekNoShows = Math.max(0, weekBooked - weekShows);
    const weekShowRate = getShowRate(weekShows, weekBooked);
    const floorAttRate =
      totalPossibleDays > 0 ? Math.round((totalAttendedDays / totalPossibleDays) * 100) : 0;

    return {
      weekBooked,
      weekShows,
      weekNoShows,
      weekShowRate,
      bestAgent,
      dayTotals,
      totalPossibleDays,
      totalAttendedDays,
      totalLateSlots,
      totalAbsentSlots,
      totalPresentOnTimeSlots,
      floorAttRate
    };
  }, [filteredAgents, selectedWeekId, agents]);

  // --- ANALYTICS & MULTI-WEEK INSIGHTS ---
  const analyticsData = useMemo(() => {
    const safeAgents = Array.isArray(agents) ? agents : INITIAL_AGENTS;
    const safeWeeks = Array.isArray(weeks) ? weeks : INITIAL_WEEKS;

    // 1. Campaign Comparison for Selected Week
    const campaignStats = Object.keys(CAMPAIGN_CONFIG).map(campName => {
      const campAgents = safeAgents.filter(a => a.campaign === campName);
      let booked = 0;
      let shows = 0;
      let attScore = 0;
      const possibleDays = campAgents.length * 5;

      campAgents.forEach(ag => {
        const p = getAgentWeekPerf(ag, selectedWeekId);
        const att = getAgentWeekAtt(ag, selectedWeekId);
        booked += p.totalBooked;
        shows += p.totalShows;
        attScore += att.scoreDays;
      });

      return {
        campaign: campName,
        shift: CAMPAIGN_CONFIG[campName].shift,
        agentCount: campAgents.length,
        booked,
        shows,
        noShows: Math.max(0, booked - shows),
        showRate: getShowRate(shows, booked),
        attRate: possibleDays > 0 ? Math.round((attScore / possibleDays) * 100) : 0
      };
    });

    // 2. Multi-Week Floor Trend across all weeks
    const multiWeekTrend = [...safeWeeks].reverse().map(w => {
      let booked = 0;
      let shows = 0;
      let attScore = 0;
      const possible = safeAgents.length * 5;

      safeAgents.forEach(ag => {
        const p = getAgentWeekPerf(ag, w.id);
        const att = getAgentWeekAtt(ag, w.id);
        booked += p.totalBooked;
        shows += p.totalShows;
        attScore += att.scoreDays;
      });

      return {
        id: w.id,
        dateRange: w.dateRange,
        booked,
        shows,
        showRate: getShowRate(shows, booked),
        attRate: possible > 0 ? Math.round((attScore / possible) * 100) : 0
      };
    });

    // 3. Coaching & Attendance Alerts for Selected Week
    const alerts = [];
    safeAgents.forEach(ag => {
      const p = getAgentWeekPerf(ag, selectedWeekId);
      const att = getAgentWeekAtt(ag, selectedWeekId);
      const reasons = [];

      if (p.totalBooked > 0 && p.showRate < 50) {
        reasons.push(`Low Show Rate (${p.showRate}%)`);
      }
      if (p.totalBooked < Math.round(weeklyBookingGoal * 0.65)) {
        reasons.push(`Below Booking Pace (${p.totalBooked}/${weeklyBookingGoal})`);
      }
      if (att.absentCount > 0) {
        reasons.push(`${att.absentCount} Absence(s)`);
      }
      if (att.lateCount >= 2) {
        reasons.push(`${att.lateCount} Late Logins`);
      }

      if (reasons.length > 0) {
        alerts.push({
          agent: ag,
          perf: p,
          att,
          reasons
        });
      }
    });

    return { campaignStats, multiWeekTrend, alerts };
  }, [agents, weeks, selectedWeekId, weeklyBookingGoal]);

  // --- UPDATERS FOR 5-DAY PERFORMANCE & ATTENDANCE ---
  const handleAdjustDayBooked = (agentId, dayKey, delta) => {
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        const currentPerf = getAgentWeekPerf(ag, selectedWeekId);
        const currentDay = currentWeekRecord.perf?.[dayKey] || { booked: 0, shows: 0 };

        const nextBooked = Math.max(0, (Number(currentDay.booked) || 0) + delta);
        const newTotalBooked = currentPerf.totalBooked - (Number(currentDay.booked) || 0) + nextBooked;
        const clampedWeekShows = Math.min(newTotalBooked, currentPerf.totalShows);

        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              weekShows: clampedWeekShows,
              perf: {
                ...currentWeekRecord.perf,
                [dayKey]: { ...currentDay, booked: nextBooked }
              }
            }
          }
        };
      })
    );
  };

  const handleSetDayBooked = (agentId, dayKey, val) => {
    const cleanVal = Math.max(0, Number(val) || 0);
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        const currentPerf = getAgentWeekPerf(ag, selectedWeekId);
        const currentDay = currentWeekRecord.perf?.[dayKey] || { booked: 0, shows: 0 };

        const newTotalBooked = currentPerf.totalBooked - (Number(currentDay.booked) || 0) + cleanVal;
        const clampedWeekShows = Math.min(newTotalBooked, currentPerf.totalShows);

        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              weekShows: clampedWeekShows,
              perf: {
                ...currentWeekRecord.perf,
                [dayKey]: { ...currentDay, booked: cleanVal }
              }
            }
          }
        };
      })
    );
  };

  const handleUpdateWeekShows = (agentId, newShowsValue) => {
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        const currentPerf = getAgentWeekPerf(ag, selectedWeekId);
        const validShows = Math.min(currentPerf.totalBooked, Math.max(0, Number(newShowsValue) || 0));

        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              weekShows: validShows
            }
          }
        };
      })
    );
  };

  const handleSetDayAttendance = (agentId, dayKey, newStatus) => {
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              att: {
                ...currentWeekRecord.att,
                [dayKey]: newStatus
              }
            }
          }
        };
      })
    );
    notify(`Updated ${dayKey.toUpperCase()} attendance to ${newStatus}`);
  };

  const handleMarkAll5DaysPresent = (agentId) => {
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              att: { d1: 'Present', d2: 'Present', d3: 'Present', d4: 'Present', d5: 'Present' }
            }
          }
        };
      })
    );
    notify('Marked all 5 days Present!');
  };

  const handleUpdateWeekNote = (agentId, noteText) => {
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              note: noteText
            }
          }
        };
      })
    );
  };

  // --- CREATE NEW 5-DAY WEEK ---
  const handleCreateNewWeek = (e) => {
    e.preventDefault();
    if (!newWeekLabel.trim()) return;
    const newId = `wk-${Date.now()}`;
    const createdWeek = {
      id: newId,
      label: `Week of ${newWeekLabel.trim()} (5 Days)`,
      dateRange: newWeekLabel.trim()
    };

    setWeeks(prev => [createdWeek, ...prev]);
    setAgents(prev =>
      prev.map(ag => ({
        ...ag,
        weeklyData: {
          ...ag.weeklyData,
          [newId]: createEmptyWeekRecord()
        }
      }))
    );
    setSelectedWeekId(newId);
    setNewWeekLabel('');
    setShowNewWeekModal(false);
    appendAuditLog(`Created new 5-day week: ${createdWeek.dateRange}`);
    notify(`Created new 5-day report: ${createdWeek.label}`);
  };

  // --- QUICK LOG APPOINTMENT / SHOW TO SPECIFIC DAY ---
  const handleQuickLogSubmit = (e) => {
    e.preventDefault();
    const bDelta = Number(quickLog.addBooked) || 0;
    const sDelta = Number(quickLog.addWeekShows) || 0;

    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== quickLog.agentId) return ag;
        const currentWeekRecord = ag.weeklyData?.[selectedWeekId] || createEmptyWeekRecord();
        const currentPerf = getAgentWeekPerf(ag, selectedWeekId);
        const currentDay = currentWeekRecord.perf?.[quickLog.dayKey] || { booked: 0, shows: 0 };

        const nextDayBooked = Math.max(0, (Number(currentDay.booked) || 0) + bDelta);
        const newTotalBooked = currentPerf.totalBooked - (Number(currentDay.booked) || 0) + nextDayBooked;
        const nextWeekShows = Math.min(newTotalBooked, Math.max(0, currentPerf.totalShows + sDelta));

        return {
          ...ag,
          weeklyData: {
            ...ag.weeklyData,
            [selectedWeekId]: {
              ...currentWeekRecord,
              weekShows: nextWeekShows,
              perf: {
                ...currentWeekRecord.perf,
                [quickLog.dayKey]: { ...currentDay, booked: nextDayBooked }
              }
            }
          }
        };
      })
    );

    const agObj = agents.find(a => a.id === quickLog.agentId);
    const dayObj = DAYS.find(d => d.key === quickLog.dayKey);
    notify(`Updated ${agObj?.name}: +${bDelta} Booked (${dayObj?.short}) & +${sDelta} End-of-Week Shows`);
    setShowQuickLogModal(false);
  };

  // --- ADD NEW AGENT ---
  const handleCreateAgent = (e) => {
    e.preventDefault();
    if (!newAgent.name.trim()) return;

    const initialWeeksMap = {};
    weeks.forEach(w => {
      initialWeeksMap[w.id] = createEmptyWeekRecord();
    });

    const created = {
      id: `ag-${Date.now()}`,
      name: newAgent.name.trim(),
      agentCode: newAgent.agentCode || `DA-${100 + agents.length + 1}`,
      campaign: newAgent.campaign,
      shift: newAgent.shift,
      phone: newAgent.phone || '01700-000000',
      weeklyData: initialWeeksMap
    };

    setAgents(prev => [...prev, created]);
    setShowAgentModal(false);
    setNewAgent({
      name: '',
      agentCode: `DA-${100 + agents.length + 2}`,
      campaign: 'Timeshare',
      shift: 'Mon–Fri (9:00 PM - 7:00 AM)',
      phone: ''
    });
    appendAuditLog(`Added agent ${created.name} (${created.agentCode}) to ${created.campaign}`);
    notify(`Added ${created.name} to Dialify Asia floor!`);
  };

  // --- UPDATE OR REMOVE AGENT ---
  const handleUpdateAgentField = (agentId, field, value) => {
    setAgents(prev =>
      prev.map(ag => {
        if (ag.id !== agentId) return ag;
        if (field === 'campaign' && CAMPAIGN_CONFIG[value]) {
          return { ...ag, campaign: value, shift: CAMPAIGN_CONFIG[value].shift };
        }
        return { ...ag, [field]: value };
      })
    );
    notify(`Updated agent ${field}`);
  };

  const handleConfirmRemoveAgent = (agent) => {
    setAgents(prev => prev.filter(ag => ag.id !== agent.id));
    if (quickLog.agentId === agent.id && agents.length > 1) {
      const remaining = agents.filter(ag => ag.id !== agent.id);
      if (remaining[0]) {
        setQuickLog(prev => ({ ...prev, agentId: remaining[0].id }));
      }
    }
    setAgentToDelete(null);
    appendAuditLog(`Removed agent ${agent.name} (${agent.agentCode}) from roster`);
    notify(`Removed ${agent.name} (${agent.agentCode}) from Dialify Asia roster`);
  };

  // --- BACKUP, RESTORE & RESET DEMO DATA ---
  const handleExportBackupJSON = () => {
    const payload = {
      version: 3,
      exportedAt: new Date().toISOString(),
      weeks,
      selectedWeekId,
      weeklyBookingGoal,
      agents
    };
    const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `Dialify_Asia_CRM_Backup_${currentWeekObj.id}.json`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
    notify('Downloaded full CRM JSON Backup!');
  };

  const handleImportBackupJSON = (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (event) => {
      try {
        const parsed = JSON.parse(event.target.result);
        if (Array.isArray(parsed.weeks) && Array.isArray(parsed.agents)) {
          setWeeks(parsed.weeks);
          setAgents(parsed.agents);
          setSelectedWeekId(parsed.selectedWeekId || parsed.weeks[0]?.id);
          if (parsed.weeklyBookingGoal) setWeeklyBookingGoal(Number(parsed.weeklyBookingGoal));
          notify('Restored CRM data from JSON backup!');
        } else {
          notify('Invalid backup file format.');
        }
      } catch (err) {
        notify('Could not parse JSON backup file.');
      }
    };
    reader.readAsText(file);
    e.target.value = '';
  };

  const handleResetDemoData = () => {
    setWeeks(INITIAL_WEEKS);
    setAgents(INITIAL_AGENTS);
    setSelectedWeekId(INITIAL_WEEKS[0].id);
    setWeeklyBookingGoal(20);
    setConfirmResetModal(false);
    appendAuditLog('Reset CRM floor data to factory demo state');
    notify('Reset floor CRM to original factory demo data!');
  };

  // --- CSV EXPORTS (PERFORMANCE & ATTENDANCE) ---
  const exportWeeklyPerformanceCSV = () => {
    const headers = [
      'Week', 'Agent ID', 'Agent Name', 'Campaign',
      'Day 1 Mon (Booked)',
      'Day 2 Tue (Booked)',
      'Day 3 Wed (Booked)',
      'Day 4 Thu (Booked)',
      'Day 5 Fri (Booked)',
      '5-Day Total Booked',
      'End-of-Week Total Shows',
      'End-of-Week No-Shows',
      'Weekly Show Rate %'
    ];

    const rows = filteredAgents.map(a => {
      const p = getAgentWeekPerf(a, selectedWeekId);
      return [
        `"${currentWeekObj.dateRange}"`,
        a.agentCode,
        `"${a.name}"`,
        `"${a.campaign}"`,
        p.days.d1.booked,
        p.days.d2.booked,
        p.days.d3.booked,
        p.days.d4.booked,
        p.days.d5.booked,
        p.totalBooked,
        p.totalShows,
        p.noShows,
        `${p.showRate}%`
      ];
    });

    const csvContent =
      'data:text/csv;charset=utf-8,' + [headers.join(','), ...rows.map(r => r.join(','))].join('\n');
    const link = document.createElement('a');
    link.setAttribute('href', encodeURI(csvContent));
    link.setAttribute('download', `Dialify_Asia_5Day_Performance_${currentWeekObj.id}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    notify('Exported 5-Day Weekly Performance CSV!');
  };

  const exportWeeklyAttendanceCSV = () => {
    const headers = [
      'Week', 'Agent ID', 'Agent Name', 'Campaign', 'Shift',
      'Day 1 (Mon)', 'Day 2 (Tue)', 'Day 3 (Wed)', 'Day 4 (Thu)', 'Day 5 (Fri)',
      'Days Attended (out of 5)', 'Weekly Attendance %', 'Supervisor Remarks'
    ];

    const rows = filteredAgents.map(a => {
      const att = getAgentWeekAtt(a, selectedWeekId);
      return [
        `"${currentWeekObj.dateRange}"`,
        a.agentCode,
        `"${a.name}"`,
        `"${a.campaign}"`,
        `"${a.shift}"`,
        att.days.d1,
        att.days.d2,
        att.days.d3,
        att.days.d4,
        att.days.d5,
        `${att.scoreDays}/5`,
        `${att.attPercent}%`,
        `"${String(att.note || '').replace(/"/g, '""')}"`
      ];
    });

    const csvContent =
      'data:text/csv;charset=utf-8,' + [headers.join(','), ...rows.map(r => r.join(','))].join('\n');
    const link = document.createElement('a');
    link.setAttribute('href', encodeURI(csvContent));
    link.setAttribute('download', `Dialify_Asia_5Day_Attendance_${currentWeekObj.id}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    notify('Exported 5-Day Weekly Attendance CSV!');
  };

  // ============================================================================
  // GATE: FULL-SCREEN ADMIN LOGIN & LOCK PORTAL WHEN UNAUTHENTICATED OR LOCKED
  // ============================================================================
  if (!currentUser || isScreenLocked) {
    return (
      <div className="min-h-screen bg-[#121316] text-slate-100 flex flex-col justify-center items-center p-4 selection:bg-[#ff5a00] selection:text-white relative overflow-hidden">
        {/* Notification Toast on Login Screen */}
        {toast && (
          <div className="fixed bottom-5 right-5 z-50 bg-[#121316] text-white px-4 py-3 rounded-xl shadow-2xl border-2 border-[#ff5a00] flex items-center gap-2.5">
            <CheckCircle2 className="w-5 h-5 text-[#ff5a00] shrink-0" />
            <span className="text-sm font-bold tracking-wide">{toast}</span>
          </div>
        )}

        {/* Ambient Orange Glow */}
        <div className="w-96 h-96 rounded-full bg-[#ff5a00]/10 blur-3xl absolute -top-24 -left-24 pointer-events-none" />
        <div className="w-96 h-96 rounded-full bg-[#ff5a00]/5 blur-3xl absolute -bottom-24 -right-24 pointer-events-none" />

        <div className="w-full max-w-md bg-[#2b303a] border-2 border-[#ff5a00] rounded-3xl shadow-2xl overflow-hidden z-10">
          {/* Top Brand Security Header */}
          <div className="bg-[#121316] px-6 py-6 border-b border-[#454d5e] text-center">
            <div className="w-14 h-14 rounded-2xl bg-[#ff5a00] flex items-center justify-center mx-auto shadow-lg shadow-[#ff5a00]/30 mb-3">
              {isScreenLocked ? (
                <Lock className="w-7 h-7 text-[#121316] stroke-[2.5]" />
              ) : (
                <Headphones className="w-7 h-7 text-[#121316] stroke-[2.5]" />
              )}
            </div>
            <div className="flex items-center justify-center gap-2">
              <h1 className="text-2xl font-black tracking-tight text-white">
                DIALIFY <span className="text-[#ff5a00]">ASIA</span>
              </h1>
              <span className="text-[10px] font-black uppercase tracking-wider px-2.5 py-0.5 rounded-full bg-[#ff5a00]/20 text-[#ff5a00] border border-[#ff5a00]/50">
                Admin Security Gate
              </span>
            </div>
            <p className="text-xs text-slate-400 mt-1.5">
              {isScreenLocked
                ? `Session Locked for ${currentUser?.name} — Enter Passcode to Resume`
                : 'Restricted 5-Day Floor CRM — Authorized Administrators Only'}
            </p>
          </div>

          <div className="p-6 space-y-5">
            {/* Private Security Notice Banner */}
            <div className="bg-[#121316] px-3.5 py-2.5 rounded-xl border border-[#3f4654] flex items-center gap-2.5">
              <Shield className="w-4 h-4 text-[#ff5a00] shrink-0" />
              <p className="text-[11px] text-slate-300 font-semibold leading-snug">
                Protected terminal. Enter your assigned administrator username and security passcode to unlock floor data.
              </p>
            </div>

            {/* Manual Login Form */}
            <form onSubmit={handleAdminLogin} className="space-y-4">
              {loginError && (
                <div className="p-3 rounded-xl bg-rose-500/15 border border-rose-500 text-rose-300 text-xs font-bold flex items-center gap-2">
                  <AlertCircle className="w-4 h-4 text-rose-400 shrink-0" />
                  <span>{loginError}</span>
                </div>
              )}

              {lockoutSeconds > 0 && (
                <div className="p-3.5 rounded-xl bg-[#121316] border-2 border-amber-500 text-center space-y-1">
                  <Clock className="w-5 h-5 text-amber-400 mx-auto animate-pulse" />
                  <p className="text-xs font-extrabold text-amber-400 uppercase tracking-wider">
                    Brute-Force Cooldown Active
                  </p>
                  <p className="text-base font-black text-white">{lockoutSeconds}s remaining</p>
                </div>
              )}

              <div>
                <label className="text-xs font-extrabold uppercase tracking-wider text-slate-300 block mb-1.5">
                  Admin Username
                </label>
                <div className="relative">
                  <Users className="w-4 h-4 text-[#ff5a00] absolute left-3.5 top-1/2 -translate-y-1/2" />
                  <input
                    type="text"
                    required
                    autoComplete="username"
                    disabled={lockoutSeconds > 0 || isScreenLocked}
                    value={loginUsername}
                    onChange={(e) => setLoginUsername(e.target.value)}
                    placeholder="Enter admin username..."
                    className="w-full pl-10 pr-4 py-2.5 bg-[#121316] text-white placeholder-slate-500 border border-[#4a5264] rounded-xl text-sm font-bold focus:outline-none focus:border-[#ff5a00] disabled:opacity-60"
                  />
                </div>
              </div>

              <div>
                <label className="text-xs font-extrabold uppercase tracking-wider text-slate-300 block mb-1.5">
                  Security Passcode
                </label>
                <div className="relative">
                  <KeyRound className="w-4 h-4 text-[#ff5a00] absolute left-3.5 top-1/2 -translate-y-1/2" />
                  <input
                    type={showPasscode ? 'text' : 'password'}
                    required
                    autoComplete="current-password"
                    disabled={lockoutSeconds > 0}
                    value={loginPasscode}
                    onChange={(e) => setLoginPasscode(e.target.value)}
                    placeholder="Enter security passcode..."
                    className="w-full pl-10 pr-10 py-2.5 bg-[#121316] text-white placeholder-slate-500 border border-[#4a5264] rounded-xl text-sm font-bold focus:outline-none focus:border-[#ff5a00] disabled:opacity-60"
                  />
                  <button
                    type="button"
                    onClick={() => setShowPasscode(!showPasscode)}
                    className="absolute right-3 top-1/2 -translate-y-1/2 text-slate-400 hover:text-[#ff5a00] cursor-pointer"
                    title={showPasscode ? 'Hide Passcode' : 'Show Passcode'}
                  >
                    {showPasscode ? <EyeOff className="w-4 h-4" /> : <Eye className="w-4 h-4" />}
                  </button>
                </div>
              </div>

              <div className="flex items-center justify-between text-xs">
                <label className="inline-flex items-center gap-2 cursor-pointer text-slate-300 font-semibold">
                  <input
                    type="checkbox"
                    checked={rememberSession}
                    onChange={(e) => setRememberSession(e.target.checked)}
                    className="rounded border-[#4a5264] bg-[#121316] text-[#ff5a00] focus:ring-[#ff5a00]"
                  />
                  <span>Keep session signed in</span>
                </label>

                {isScreenLocked && (
                  <button
                    type="button"
                    onClick={handleSignOut}
                    className="text-rose-400 hover:underline font-bold cursor-pointer"
                  >
                    Switch Admin Account
                  </button>
                )}
              </div>

              <button
                type="submit"
                disabled={lockoutSeconds > 0}
                className="w-full py-3 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] disabled:opacity-50 text-[#121316] font-black text-sm shadow-lg shadow-[#ff5a00]/25 transition cursor-pointer flex items-center justify-center gap-2"
              >
                <Unlock className="w-4 h-4 stroke-[2.5]" />
                <span>{isScreenLocked ? 'Unlock Floor Terminal' : 'Authorize & Enter Floor CRM'}</span>
              </button>
            </form>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-[#2b303a] text-slate-100 font-sans pb-14 selection:bg-[#ff5a00] selection:text-white">
      {/* HIDDEN FILE INPUT FOR JSON RESTORE */}
      <input
        ref={fileInputRef}
        type="file"
        accept=".json"
        onChange={handleImportBackupJSON}
        className="hidden"
      />

      {/* NOTIFICATION TOAST */}
      {toast && (
        <div className="fixed bottom-5 right-5 z-50 bg-[#121316] text-white px-4 py-3 rounded-xl shadow-2xl border-2 border-[#ff5a00] flex items-center gap-2.5">
          <CheckCircle2 className="w-5 h-5 text-[#ff5a00] shrink-0" />
          <span className="text-sm font-bold tracking-wide">{toast}</span>
        </div>
      )}

      {/* TOP HEADER BAR WITH 5-DAY WEEK SELECTOR, ADMIN BADGE & SECURITY LOCK CONTROLS */}
      <header className="bg-[#121316] text-white border-b-2 border-[#ff5a00] sticky top-0 z-30 shadow-xl">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 py-3.5 flex flex-col xl:flex-row xl:items-center xl:justify-between gap-3.5">
          {/* Brand Identity & Active Admin Badge */}
          <div className="flex items-center justify-between gap-3 flex-wrap">
            <div className="flex items-center gap-3">
              <div className="w-11 h-11 rounded-2xl bg-[#ff5a00] flex items-center justify-center shadow-lg shadow-[#ff5a00]/30 shrink-0">
                <Headphones className="w-6 h-6 text-[#121316] stroke-[2.5]" />
              </div>
              <div>
                <div className="flex items-center gap-2 flex-wrap">
                  <h1 className="text-xl font-black tracking-tight text-white">
                    DIALIFY <span className="text-[#ff5a00]">ASIA</span>
                  </h1>
                  <span className="text-[11px] font-extrabold uppercase tracking-wider px-2.5 py-0.5 rounded-full bg-[#ff5a00]/20 text-[#ff5a00] border border-[#ff5a00]/50">
                    5-Day Weekly Floor CRM
                  </span>
                  <span
                    className="inline-flex items-center gap-1 text-[10px] font-bold px-2 py-0.5 rounded-md bg-emerald-500/15 text-emerald-400 border border-emerald-500/30"
                    title="All edits automatically persist in browser localStorage"
                  >
                    <Check className="w-3 h-3" />
                    Auto-Saved ({lastSavedTime})
                  </span>
                </div>
                <p className="text-xs text-slate-400">
                  Weekly Agent Appointments Booked vs. Shows (5-Day Workweek), Attendance & Analytics
                </p>
              </div>
            </div>

            {/* Logged-in Admin Session Pill */}
            <div className="flex items-center gap-2 bg-[#1c1f26] px-3 py-1.5 rounded-xl border border-[#3f4654]">
              <Shield className="w-4 h-4 text-[#ff5a00] shrink-0" />
              <div className="text-xs">
                <div className="flex items-center gap-1.5">
                  <span className="font-bold text-white">{currentUser.name}</span>
                  <span className="text-[10px] font-black px-1.5 py-0.5 rounded bg-[#ff5a00] text-[#121316]">
                    {currentUser.role}
                  </span>
                </div>
              </div>
              {isSuperAdmin && (
                <button
                  onClick={() => setShowSecurityModal(true)}
                  className="ml-1 p-1.5 rounded-lg bg-[#2b303a] hover:bg-[#ff5a00] text-slate-200 hover:text-[#121316] transition cursor-pointer"
                  title="Admin Security & Audit Log"
                >
                  <KeyRound className="w-3.5 h-3.5" />
                </button>
              )}
              <button
                onClick={handleLockScreen}
                className="p-1.5 rounded-lg bg-[#2b303a] hover:bg-amber-500/20 text-amber-400 transition cursor-pointer"
                title="Lock CRM Screen Immediately"
              >
                <Lock className="w-3.5 h-3.5" />
              </button>
              <button
                onClick={handleSignOut}
                className="p-1.5 rounded-lg bg-[#2b303a] hover:bg-rose-500/20 text-rose-400 transition cursor-pointer"
                title="Sign Out"
              >
                <LogOut className="w-3.5 h-3.5" />
              </button>
            </div>
          </div>

          {/* Week Selector & Action Buttons */}
          <div className="flex flex-wrap items-center gap-2">
            {/* Active 5-Day Week Dropdown */}
            <div className="flex items-center bg-[#2b303a] border border-[#454d5e] rounded-xl px-3 py-1.5">
              <CalendarDays className="w-4 h-4 text-[#ff5a00] mr-2 shrink-0" />
              <div className="flex flex-col">
                <span className="text-[10px] uppercase tracking-wider text-slate-300 font-bold">
                  Selected 5-Day Week
                </span>
                <select
                  value={selectedWeekId}
                  onChange={(e) => setSelectedWeekId(e.target.value)}
                  className="bg-transparent text-white text-xs sm:text-sm font-bold focus:outline-none cursor-pointer pr-2"
                >
                  {weeks.map(w => (
                    <option key={w.id} value={w.id} className="bg-[#121316] text-white">
                      {w.label}
                    </option>
                  ))}
                </select>
              </div>
            </div>

            <button
              onClick={() => setShowNewWeekModal(true)}
              className="inline-flex items-center gap-1.5 px-3 py-2.5 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] font-extrabold text-xs sm:text-sm shadow-md shadow-[#ff5a00]/20 transition cursor-pointer"
            >
              <Plus className="w-4 h-4 stroke-[2.5]" />
              <span>New Week</span>
            </button>

            <button
              onClick={() => setShowQuickLogModal(true)}
              className="inline-flex items-center gap-1.5 px-3 py-2.5 rounded-xl bg-[#2b303a] hover:bg-[#353b48] text-[#ff5a00] border border-[#ff5a00] font-bold text-xs sm:text-sm shadow-xs transition cursor-pointer"
            >
              <PhoneCall className="w-4 h-4" />
              <span>Log Day Entry</span>
            </button>

            {isSuperAdmin && (
              <button
                onClick={() => setShowAgentModal(true)}
                className="inline-flex items-center gap-1.5 px-3 py-2.5 rounded-xl bg-[#2b303a] hover:bg-[#353b48] text-slate-200 border border-[#454d5e] font-semibold text-xs sm:text-sm transition cursor-pointer"
              >
                <UserPlus className="w-4 h-4 text-[#ff5a00]" />
                <span>Add Agent</span>
              </button>
            )}

            {/* Backup / Restore / Reset Controls (Super Admin Only) */}
            {isSuperAdmin && (
              <div className="flex items-center gap-1 bg-[#1c1f26] p-1 rounded-xl border border-[#3f4654]">
                <button
                  onClick={handleExportBackupJSON}
                  className="p-2 rounded-lg text-slate-300 hover:text-[#ff5a00] hover:bg-[#2b303a] transition cursor-pointer"
                  title="Download JSON Backup"
                >
                  <Download className="w-4 h-4" />
                </button>
                <button
                  onClick={() => fileInputRef.current?.click()}
                  className="p-2 rounded-lg text-slate-300 hover:text-[#ff5a00] hover:bg-[#2b303a] transition cursor-pointer"
                  title="Restore from JSON Backup"
                >
                  <Upload className="w-4 h-4" />
                </button>
                <button
                  onClick={() => setConfirmResetModal(true)}
                  className="p-2 rounded-lg text-slate-300 hover:text-rose-400 hover:bg-[#2b303a] transition cursor-pointer"
                  title="Reset to Factory Demo Data"
                >
                  <RotateCcw className="w-4 h-4" />
                </button>
              </div>
            )}
          </div>
        </div>
      </header>

      {/* MAIN CONTENT (STORMY GREY ENVIRONMENT) */}
      <main className="max-w-7xl mx-auto px-4 sm:px-6 pt-6 space-y-6">
        {/* CAMPAIGN SCHEDULE & SHIFT TIMING BAR */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-3">
          {Object.entries(CAMPAIGN_CONFIG).map(([campName, cfg]) => {
            const isSelected = campaignFilter === campName;
            const campAgentsCount = agents.filter(a => a.campaign === campName).length;
            return (
              <div
                key={campName}
                onClick={() => setCampaignFilter(isSelected ? 'All Campaigns' : campName)}
                className={`p-3.5 rounded-xl border transition cursor-pointer flex items-center justify-between ${
                  isSelected
                    ? 'bg-[#121316] border-2 border-[#ff5a00] shadow-md shadow-[#ff5a00]/20'
                    : 'bg-[#121316]/90 border-[#3f4654] hover:border-[#ff5a00]/60'
                }`}
              >
                <div>
                  <div className="flex items-center gap-2">
                    <span className="w-2.5 h-2.5 rounded-full bg-[#ff5a00]" />
                    <span className="text-xs sm:text-sm font-extrabold text-white">{campName}</span>
                    <span className="text-[10px] font-bold px-2 py-0.5 rounded-full bg-[#2b303a] text-slate-300">
                      {campAgentsCount} Agents
                    </span>
                  </div>
                  <p className="text-xs text-[#ff5a00] font-bold mt-1">{cfg.shift}</p>
                </div>
                <span className="text-[10px] font-bold px-2.5 py-1 rounded-lg bg-[#2b303a] text-slate-300 border border-[#454d5e]">
                  {isSelected ? 'Active Filter ✕' : 'Click to Filter'}
                </span>
              </div>
            );
          })}
        </div>

        {/* MAIN 4-TAB SWITCHER (PERFORMANCE, ATTENDANCE, ANALYTICS, MANAGE AGENTS) */}
        <div className="bg-[#353b48] p-4 rounded-2xl border border-[#4a5264] shadow-lg flex flex-col xl:flex-row xl:items-center justify-between gap-4">
          <div className="flex flex-wrap items-center gap-1.5 bg-[#121316] p-1.5 rounded-xl border border-[#2b303a]">
            <button
              onClick={() => setActiveTab('performance')}
              className={`flex items-center gap-2 px-4 py-2.5 rounded-xl text-xs sm:text-sm font-extrabold transition cursor-pointer ${
                activeTab === 'performance'
                  ? 'bg-[#ff5a00] text-[#121316] shadow-md shadow-[#ff5a00]/30'
                  : 'text-slate-300 hover:text-[#ff5a00]'
              }`}
            >
              <BarChart3 className="w-4 h-4" />
              <span>1. Performance (5-Day)</span>
            </button>

            <button
              onClick={() => setActiveTab('attendance')}
              className={`flex items-center gap-2 px-4 py-2.5 rounded-xl text-xs sm:text-sm font-extrabold transition cursor-pointer ${
                activeTab === 'attendance'
                  ? 'bg-[#ff5a00] text-[#121316] shadow-md shadow-[#ff5a00]/30'
                  : 'text-slate-300 hover:text-[#ff5a00]'
              }`}
            >
              <UserCheck className="w-4 h-4" />
              <span>2. Attendance Sheet</span>
            </button>

            <button
              onClick={() => setActiveTab('analytics')}
              className={`flex items-center gap-2 px-4 py-2.5 rounded-xl text-xs sm:text-sm font-extrabold transition cursor-pointer ${
                activeTab === 'analytics'
                  ? 'bg-[#ff5a00] text-[#121316] shadow-md shadow-[#ff5a00]/30'
                  : 'text-slate-300 hover:text-[#ff5a00]'
              }`}
            >
              <Activity className="w-4 h-4" />
              <span>3. Floor Analytics & Coaching</span>
              {analyticsData.alerts.length > 0 && (
                <span
                  className={`px-2 py-0.5 rounded-full text-[11px] font-black ${
                    activeTab === 'analytics'
                      ? 'bg-[#121316] text-[#ff5a00]'
                      : 'bg-rose-500/20 text-rose-400 border border-rose-500/40'
                  }`}
                >
                  {analyticsData.alerts.length}
                </span>
              )}
            </button>

            <button
              onClick={() => {
                if (!isSuperAdmin) {
                  notify('Manage Agents tab requires Super Admin privileges.');
                  return;
                }
                setActiveTab('agents');
              }}
              className={`flex items-center gap-2 px-4 py-2.5 rounded-xl text-xs sm:text-sm font-extrabold transition cursor-pointer ${
                activeTab === 'agents'
                  ? 'bg-[#ff5a00] text-[#121316] shadow-md shadow-[#ff5a00]/30'
                  : 'text-slate-300 hover:text-[#ff5a00]'
              }`}
            >
              <Users className="w-4 h-4" />
              <span>4. Manage Agents</span>
              {!isSuperAdmin && <Lock className="w-3.5 h-3.5 text-amber-400" />}
              <span
                className={`px-2 py-0.5 rounded-full text-[11px] font-black ${
                  activeTab === 'agents'
                    ? 'bg-[#121316] text-[#ff5a00]'
                    : 'bg-[#2b303a] text-[#ff5a00]'
                }`}
              >
                {agents.length}
              </span>
            </button>
          </div>

          {/* Search, Campaign Filter, Sort & Context CSV Export */}
          <div className="flex flex-wrap items-center gap-2">
            <div className="relative flex-1 sm:w-48">
              <Search className="w-4 h-4 text-[#ff5a00] absolute left-3 top-1/2 -translate-y-1/2" />
              <input
                type="text"
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
                placeholder="Search agent or ID..."
                className="w-full pl-9 pr-3 py-2 bg-[#121316] text-white placeholder-slate-400 border border-[#4a5264] rounded-xl text-xs sm:text-sm focus:outline-none focus:ring-2 focus:ring-[#ff5a00]"
              />
            </div>

            <select
              value={campaignFilter}
              onChange={(e) => setCampaignFilter(e.target.value)}
              className="px-3 py-2 bg-[#121316] border border-[#4a5264] rounded-xl text-xs sm:text-sm font-semibold text-slate-200 focus:outline-none focus:border-[#ff5a00] cursor-pointer"
            >
              {CAMPAIGNS.map(c => (
                <option key={c} value={c} className="bg-[#121316] text-white">{c}</option>
              ))}
            </select>

            {activeTab === 'performance' && (
              <div className="flex items-center bg-[#121316] border border-[#4a5264] rounded-xl px-2.5 py-1.5">
                <ArrowUpDown className="w-3.5 h-3.5 text-[#ff5a00] mr-1.5 shrink-0" />
                <select
                  value={sortBy}
                  onChange={(e) => setSortBy(e.target.value)}
                  className="bg-transparent text-xs sm:text-sm font-semibold text-slate-200 focus:outline-none cursor-pointer"
                >
                  <option value="default" className="bg-[#121316]">Sort: Default Roster</option>
                  <option value="shows-desc" className="bg-[#121316]">Sort: Highest Shows</option>
                  <option value="booked-desc" className="bg-[#121316]">Sort: Highest Booked</option>
                  <option value="rate-desc" className="bg-[#121316]">Sort: Highest Show Rate %</option>
                  <option value="name-asc" className="bg-[#121316]">Sort: Agent Name (A–Z)</option>
                </select>
              </div>
            )}

            {activeTab === 'performance' ? (
              <button
                onClick={exportWeeklyPerformanceCSV}
                className="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] text-xs sm:text-sm font-extrabold transition cursor-pointer shadow-md shadow-[#ff5a00]/20"
              >
                <Download className="w-4 h-4 stroke-[2.5]" />
                <span>Export CSV</span>
              </button>
            ) : activeTab === 'attendance' ? (
              <button
                onClick={exportWeeklyAttendanceCSV}
                className="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] text-xs sm:text-sm font-extrabold transition cursor-pointer shadow-md shadow-[#ff5a00]/20"
              >
                <Download className="w-4 h-4 stroke-[2.5]" />
                <span>Export Attendance CSV</span>
              </button>
            ) : isSuperAdmin ? (
              <button
                onClick={() => setShowAgentModal(true)}
                className="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] text-xs sm:text-sm font-extrabold transition cursor-pointer shadow-md shadow-[#ff5a00]/20"
              >
                <UserPlus className="w-4 h-4 stroke-[2.5]" />
                <span>+ New Agent</span>
              </button>
            ) : null}
          </div>
        </div>

        {/* =====================================================================
            TAB 1: WEEKLY PERFORMANCE REPORT (5-DAY WORKWEEK: BOOKED VS SHOWS)
           ===================================================================== */}
        {activeTab === 'performance' && (
          <div className="space-y-6">
            {/* 4 WEEKLY PERFORMANCE KPI CARDS */}
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    Weekly Appointments Booked
                  </span>
                  <p className="text-3xl font-extrabold text-white mt-1">{weeklySummary.weekBooked}</p>
                  <p className="text-xs font-bold text-[#ff5a00] mt-1">
                    {currentWeekObj.dateRange} (5 Days)
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00]/15 text-[#ff5a00] border border-[#ff5a00]/30 flex items-center justify-center">
                  <Calendar className="w-6 h-6" />
                </div>
              </div>

              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    Weekly Client Shows
                  </span>
                  <p className="text-3xl font-extrabold text-[#ff5a00] mt-1">{weeklySummary.weekShows}</p>
                  <p className="text-xs font-medium text-slate-400 mt-1">
                    {weeklySummary.weekNoShows} Weekly No-Shows
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00]/15 text-[#ff5a00] border border-[#ff5a00]/30 flex items-center justify-center">
                  <CheckCircle2 className="w-6 h-6" />
                </div>
              </div>

              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    Weekly Show Rate %
                  </span>
                  <div className="flex items-baseline gap-2 mt-1">
                    <p className="text-3xl font-extrabold text-white">{weeklySummary.weekShowRate}%</p>
                    <span className="text-xs font-bold text-[#ff5a00]">(Shows ÷ Booked)</span>
                  </div>
                  <p className="text-xs font-medium text-slate-400 mt-1">
                    5-Day Floor Conversion
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00]/15 text-[#ff5a00] border border-[#ff5a00]/30 flex items-center justify-center">
                  <TrendingUp className="w-6 h-6" />
                </div>
              </div>

              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    Top Weekly Performer
                  </span>
                  <p className="text-lg font-extrabold text-white mt-1 truncate">
                    {weeklySummary.bestAgent ? weeklySummary.bestAgent.name : '—'}
                  </p>
                  <p className="text-xs font-bold text-[#ff5a00] mt-1">
                    {weeklySummary.bestAgent
                      ? `${weeklySummary.bestAgent.weekShows} Shows / ${weeklySummary.bestAgent.weekBooked} Booked (${weeklySummary.bestAgent.showRate}%)`
                      : 'No data'}
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00] text-[#121316] flex items-center justify-center shadow-md shadow-[#ff5a00]/30">
                  <Award className="w-6 h-6 stroke-[2.5]" />
                </div>
              </div>
            </div>

            {/* 5-DAY WEEKLY PERFORMANCE TABLE */}
            <div className="bg-[#353b48] rounded-2xl border border-[#4a5264] shadow-xl overflow-hidden">
              <div className="px-6 py-4 border-b border-[#4a5264] bg-[#252932] flex flex-col lg:flex-row lg:items-center justify-between gap-3">
                <div>
                  <div className="flex items-center gap-2 flex-wrap">
                    <h2 className="text-base font-extrabold text-white">
                      5-Day Weekly Agent Performance Report — {currentWeekObj.dateRange}
                    </h2>
                    <span className="text-xs font-extrabold px-2.5 py-0.5 rounded-full bg-[#ff5a00] text-[#121316]">
                      Daily Booked (Mon–Fri) → End-of-Week Shows Result
                    </span>
                  </div>
                  <p className="text-xs text-slate-300 mt-0.5">
                    Click any agent's name to view their <strong className="text-[#ff5a00]">Multi-Week Profile</strong>. Log <strong className="text-[#ff5a00]">Appointments Booked</strong> Day 1–Day 5, then enter <strong className="text-[#ff5a00]">End-of-Week Shows</strong>.
                  </p>
                </div>

                <div className="flex flex-wrap items-center gap-4 text-xs font-bold">
                  <div className="flex items-center gap-2 bg-[#121316] px-3 py-1.5 rounded-xl border border-[#454d5e]">
                    <Target className="w-3.5 h-3.5 text-[#ff5a00]" />
                    <span className="text-slate-300">Agent 5-Day Goal:</span>
                    <input
                      type="number"
                      min="1"
                      max="100"
                      value={weeklyBookingGoal}
                      onChange={(e) => setWeeklyBookingGoal(Math.max(1, Number(e.target.value) || 20))}
                      className="w-12 text-center bg-[#2b303a] text-[#ff5a00] font-black rounded-md py-0.5 focus:outline-none border border-[#ff5a00]/40"
                    />
                    <span className="text-slate-400">Booked</span>
                  </div>

                  <span className="inline-flex items-center gap-1.5 text-[#ff5a00]">
                    <span className="w-2.5 h-2.5 rounded-full bg-[#ff5a00]" /> 70%+ Target
                  </span>
                  <span className="inline-flex items-center gap-1.5 text-amber-400">
                    <span className="w-2.5 h-2.5 rounded-full bg-amber-400" /> 50–69%
                  </span>
                  <span className="inline-flex items-center gap-1.5 text-rose-400">
                    <span className="w-2.5 h-2.5 rounded-full bg-rose-500" /> &lt;50%
                  </span>
                </div>
              </div>

              <div className="overflow-x-auto">
                <table className="w-full text-left border-collapse">
                  <thead>
                    <tr className="bg-[#121316] border-b-2 border-[#ff5a00] text-[11px] font-extrabold uppercase tracking-wider text-slate-300">
                      <th className="py-3.5 px-4 min-w-[210px]">Agent & Campaign</th>
                      {DAYS.map((day, idx) => {
                        const specificDayName =
                          campaignFilter !== 'All Campaigns' && CAMPAIGN_CONFIG[campaignFilter]
                            ? `${day.label} (${CAMPAIGN_CONFIG[campaignFilter].days[idx]})`
                            : `${day.label} (${day.subLabel})`;
                        return (
                          <th key={day.key} className="py-3.5 px-2.5 text-center border-l border-[#2b303a] min-w-[105px]">
                            <div className="text-white">{specificDayName}</div>
                            <div className="text-[10px] font-bold text-[#ff5a00] mt-0.5">Booked Only</div>
                          </th>
                        );
                      })}
                      <th className="py-3.5 px-3 text-center bg-[#1c1f26] border-l-2 border-[#ff5a00]/60 text-white min-w-[120px]">
                        <div>5-Day Total</div>
                        <div className="text-[10px] text-[#ff5a00]">Booked (Goal: {weeklyBookingGoal})</div>
                      </th>
                      <th className="py-3.5 px-3 text-center bg-[#ff5a00]/15 border-l-2 border-[#ff5a00] text-white min-w-[155px]">
                        <div className="text-[#ff5a00]">End-of-Week Shows</div>
                        <div className="text-[10px] text-slate-200 font-bold">(Enter Weekly Result)</div>
                      </th>
                      <th className="py-3.5 px-3 text-center bg-[#181a20] border-l border-[#2b303a] text-slate-300">
                        No-Shows
                      </th>
                      <th className="py-3.5 px-4 bg-[#121316] border-l border-[#2b303a] min-w-[170px] text-[#ff5a00]">
                        Weekly Show Rate %
                      </th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-[#454d5e] text-sm">
                    {filteredAgents.map(agent => {
                      const perf = getAgentWeekPerf(agent, selectedWeekId);
                      const badge = getShowRateBadge(perf.showRate);
                      const rank = agentRankMap[agent.id];
                      const metBookingGoal = perf.totalBooked >= weeklyBookingGoal;

                      return (
                        <tr key={agent.id} className="hover:bg-[#3e4554] transition">
                          <td className="py-3.5 px-4">
                            <div className="flex items-center gap-2.5">
                              <div className="relative">
                                <div className="w-9 h-9 rounded-xl bg-[#ff5a00] text-[#121316] font-black text-xs flex items-center justify-center shrink-0 shadow-sm">
                                  {String(agent.name || 'A').split(' ').map(n => n[0]).join('')}
                                </div>
                                {rank && rank <= 3 && (
                                  <span
                                    className={`absolute -top-1.5 -right-1.5 text-[10px] font-black px-1.5 rounded-full border ${
                                      rank === 1
                                        ? 'bg-amber-400 text-[#121316] border-white'
                                        : rank === 2
                                        ? 'bg-slate-200 text-[#121316] border-white'
                                        : 'bg-amber-700 text-white border-amber-300'
                                    }`}
                                    title={`Rank #${rank} in Shows this week`}
                                  >
                                    #{rank}
                                  </span>
                                )}
                              </div>
                              <div>
                                <div className="flex items-center gap-1.5 flex-wrap">
                                  <button
                                    onClick={() => setSelectedAgentProfile(agent)}
                                    className="font-bold text-white hover:text-[#ff5a00] transition cursor-pointer inline-flex items-center gap-1 text-left"
                                    title="View Multi-Week Agent Profile"
                                  >
                                    <span>{agent.name}</span>
                                    <Eye className="w-3.5 h-3.5 text-slate-400 hover:text-[#ff5a00]" />
                                  </button>
                                  <span className="text-[10px] font-bold px-1.5 py-0.5 rounded bg-[#121316] text-[#ff5a00] border border-[#ff5a00]/40">
                                    {agent.agentCode}
                                  </span>
                                </div>
                                <p className="text-xs text-[#ff5a00] font-bold">{agent.campaign}</p>
                                <p className="text-[11px] text-slate-300">{agent.shift}</p>
                              </div>
                            </div>
                          </td>

                          {DAYS.map((day, idx) => {
                            const dVal = perf.days?.[day.key] || { booked: 0 };
                            const agentDayName = CAMPAIGN_CONFIG[agent.campaign]?.days[idx] || day.short;
                            return (
                              <td key={day.key} className="py-3 px-2 border-l border-[#454d5e] text-center align-middle">
                                <span className="block text-[10px] font-bold text-slate-400 mb-1 uppercase">
                                  {agentDayName}
                                </span>
                                <div className="inline-flex items-center justify-between w-24 px-1.5 py-1 rounded-xl bg-[#252932] border border-[#4a5264] text-xs">
                                  <button
                                    onClick={() => handleAdjustDayBooked(agent.id, day.key, -1)}
                                    className="w-5 h-5 rounded-md bg-[#121316] hover:bg-slate-800 text-slate-300 font-bold flex items-center justify-center cursor-pointer"
                                    title={`Decrease ${agentDayName} Booked`}
                                  >
                                    -
                                  </button>
                                  <input
                                    type="number"
                                    min="0"
                                    value={dVal.booked}
                                    onChange={(e) => handleSetDayBooked(agent.id, day.key, e.target.value)}
                                    className="w-9 text-center font-extrabold text-sm text-white bg-transparent focus:outline-none"
                                    title="Appointments Booked"
                                  />
                                  <button
                                    onClick={() => handleAdjustDayBooked(agent.id, day.key, 1)}
                                    className="w-5 h-5 rounded-md bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] font-black flex items-center justify-center cursor-pointer"
                                    title={`Increase ${agentDayName} Booked`}
                                  >
                                    +
                                  </button>
                                </div>
                              </td>
                            );
                          })}

                          <td className="py-3.5 px-3 text-center bg-[#252932] border-l-2 border-[#ff5a00]/60">
                            <div className="flex items-center justify-center gap-1">
                              <span className="text-xl font-black text-white">{perf.totalBooked}</span>
                              {metBookingGoal && (
                                <Award
                                  className="w-4 h-4 text-[#ff5a00]"
                                  title={`Met weekly goal of ${weeklyBookingGoal} Booked!`}
                                />
                              )}
                            </div>
                            <span className="block text-[10px] font-bold text-[#ff5a00] uppercase">
                              {metBookingGoal ? 'Goal Met!' : `${perf.totalBooked}/${weeklyBookingGoal} Goal`}
                            </span>
                          </td>

                          <td className="py-3 px-3 text-center bg-[#121316]/80 border-l-2 border-[#ff5a00]">
                            <div className="inline-flex items-center justify-between gap-1.5 px-2.5 py-1.5 rounded-xl bg-[#252932] border-2 border-[#ff5a00] shadow-md">
                              <button
                                onClick={() => handleUpdateWeekShows(agent.id, perf.totalShows - 1)}
                                className="w-6 h-6 rounded-lg bg-[#121316] hover:bg-slate-800 text-slate-200 font-extrabold flex items-center justify-center cursor-pointer"
                                title="Decrease End-of-Week Shows"
                              >
                                -
                              </button>
                              <input
                                type="number"
                                min="0"
                                max={perf.totalBooked}
                                value={perf.totalShows}
                                onChange={(e) => handleUpdateWeekShows(agent.id, e.target.value)}
                                className="w-12 text-center font-black text-base text-[#ff5a00] focus:outline-none bg-transparent"
                                title="Type End-of-Week Confirmed Shows"
                              />
                              <button
                                onClick={() => handleUpdateWeekShows(agent.id, perf.totalShows + 1)}
                                className="w-6 h-6 rounded-lg bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] font-black flex items-center justify-center cursor-pointer"
                                title="Increase End-of-Week Shows"
                              >
                                +
                              </button>
                            </div>
                            <span className="block text-[10px] font-bold text-slate-300 mt-1">
                              Out of {perf.totalBooked} Booked
                            </span>
                          </td>

                          <td className="py-3.5 px-3 text-center bg-[#252932]/60 border-l border-[#454d5e]">
                            <span className="text-sm font-bold text-slate-200 bg-[#121316] px-2.5 py-1 rounded-lg border border-[#4a5264]">
                              {perf.noShows}
                            </span>
                          </td>

                          <td className="py-3.5 px-4 bg-[#252932] border-l border-[#454d5e]">
                            <div className="flex items-center justify-between mb-1">
                              <span className={`text-xs font-extrabold px-2.5 py-0.5 rounded-full border ${badge.bg}`}>
                                {perf.showRate}%
                              </span>
                              <span className="text-[11px] font-bold text-slate-300">
                                {perf.totalShows}/{perf.totalBooked}
                              </span>
                            </div>
                            <div className="w-full h-2 bg-[#121316] rounded-full overflow-hidden">
                              <div
                                className={`h-full rounded-full transition-all duration-300 ${badge.bar}`}
                                style={{ width: `${Math.min(100, perf.showRate)}%` }}
                              />
                            </div>
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>

                  <tfoot>
                    <tr className="bg-[#121316] text-white font-bold text-xs border-t-2 border-[#ff5a00]">
                      <td className="py-3.5 px-4 text-[#ff5a00] font-black">
                        FLOOR 5-DAY TOTALS ({filteredAgents.length} Agents)
                      </td>
                      {DAYS.map(day => (
                        <td key={day.key} className="py-3 px-2 text-center border-l border-[#2b303a]">
                          <span className="text-white text-sm font-extrabold">
                            {weeklySummary.dayTotals[day.key].booked}
                          </span>
                          <span className="block text-[10px] text-slate-400 font-normal">Booked</span>
                        </td>
                      ))}
                      <td className="py-3.5 px-3 text-center border-l-2 border-[#ff5a00]/60 text-base text-white font-black">
                        {weeklySummary.weekBooked}
                      </td>
                      <td className="py-3.5 px-3 text-center border-l-2 border-[#ff5a00] text-base text-[#ff5a00] font-black">
                        {weeklySummary.weekShows} Shows
                      </td>
                      <td className="py-3.5 px-3 text-center border-l border-[#2b303a] text-slate-300">
                        {weeklySummary.weekNoShows}
                      </td>
                      <td className="py-3.5 px-4 border-l border-[#2b303a] text-[#ff5a00] text-sm font-black">
                        {weeklySummary.weekShowRate}% Weekly Show Rate
                      </td>
                    </tr>
                  </tfoot>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* =====================================================================
            TAB 2: SEPARATE WEEKLY ATTENDANCE TAB (5-DAY WORKWEEK)
           ===================================================================== */}
        {activeTab === 'attendance' && (
          <div className="space-y-6">
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    5-Day Floor Attendance Rate
                  </span>
                  <p className="text-3xl font-extrabold text-white mt-1">{weeklySummary.floorAttRate}%</p>
                  <p className="text-xs font-bold text-[#ff5a00] mt-1">
                    {weeklySummary.totalAttendedDays} of {weeklySummary.totalPossibleDays} agent-days
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00]/15 text-[#ff5a00] border border-[#ff5a00]/30 flex items-center justify-center">
                  <UserCheck className="w-6 h-6" />
                </div>
              </div>

              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    On-Time Present Days
                  </span>
                  <p className="text-3xl font-extrabold text-[#ff5a00] mt-1">
                    {weeklySummary.totalPresentOnTimeSlots}
                  </p>
                  <p className="text-xs font-medium text-slate-400 mt-1">
                    Across {currentWeekObj.dateRange}
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00]/15 text-[#ff5a00] border border-[#ff5a00]/30 flex items-center justify-center">
                  <CheckCircle2 className="w-6 h-6" />
                </div>
              </div>

              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    Weekly Late Logins
                  </span>
                  <p className="text-3xl font-extrabold text-amber-400 mt-1">{weeklySummary.totalLateSlots}</p>
                  <p className="text-xs font-medium text-slate-400 mt-1">
                    Instances across 5-day week
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-amber-500/15 text-amber-400 border border-amber-500/30 flex items-center justify-center">
                  <Clock className="w-6 h-6" />
                </div>
              </div>

              <div className="bg-[#121316] p-5 rounded-2xl border-l-4 border-[#ff5a00] border-t border-r border-b border-[#3f4654] shadow-lg flex items-center justify-between">
                <div>
                  <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                    Weekly Absences
                  </span>
                  <p className="text-3xl font-extrabold text-rose-400 mt-1">{weeklySummary.totalAbsentSlots}</p>
                  <p className="text-xs font-medium text-slate-400 mt-1">
                    Missed shifts this week
                  </p>
                </div>
                <div className="w-12 h-12 rounded-2xl bg-rose-500/15 text-rose-400 border border-rose-500/30 flex items-center justify-center">
                  <XCircle className="w-6 h-6" />
                </div>
              </div>
            </div>

            <div className="bg-[#353b48] rounded-2xl border border-[#4a5264] shadow-xl overflow-hidden">
              <div className="px-6 py-4 border-b border-[#4a5264] bg-[#252932] flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                <div>
                  <div className="flex items-center gap-2">
                    <h2 className="text-base font-extrabold text-white">
                      5-Day Weekly Attendance Sheet — {currentWeekObj.dateRange}
                    </h2>
                    <span className="text-xs font-extrabold px-2.5 py-0.5 rounded-full bg-[#ff5a00] text-[#121316]">
                      Separate Attendance Roster
                    </span>
                  </div>
                  <p className="text-xs text-slate-300 mt-0.5">
                    Select any day's status (<strong className="text-[#ff5a00]">Present</strong>, <strong className="text-amber-300">Late</strong>, <strong className="text-sky-300">Half-Day</strong>, <strong className="text-rose-400">Absent</strong>) to update the agent's 5-day attendance score.
                  </p>
                </div>
              </div>

              <div className="overflow-x-auto">
                <table className="w-full text-left border-collapse">
                  <thead>
                    <tr className="bg-[#121316] border-b-2 border-[#ff5a00] text-[11px] font-extrabold uppercase tracking-wider text-slate-300">
                      <th className="py-3.5 px-4 min-w-[200px]">Agent, Campaign & Shift</th>
                      {DAYS.map((day, idx) => {
                        const specificDayName =
                          campaignFilter !== 'All Campaigns' && CAMPAIGN_CONFIG[campaignFilter]
                            ? `${day.label} (${CAMPAIGN_CONFIG[campaignFilter].days[idx]})`
                            : `${day.label} (${day.subLabel})`;
                        return (
                          <th key={day.key} className="py-3.5 px-3 text-center border-l border-[#2b303a] min-w-[125px] text-white">
                            {specificDayName}
                          </th>
                        );
                      })}
                      <th className="py-3.5 px-4 text-center bg-[#1c1f26] border-l-2 border-[#ff5a00] text-[#ff5a00] min-w-[140px]">
                        5-Day Score
                      </th>
                      <th className="py-3.5 px-4 border-l border-[#2b303a] min-w-[220px]">
                        Weekly Attendance Note
                      </th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-[#454d5e] text-sm">
                    {filteredAgents.map(agent => {
                      const att = getAgentWeekAtt(agent, selectedWeekId);

                      return (
                        <tr key={agent.id} className="hover:bg-[#3e4554] transition">
                          <td className="py-3.5 px-4">
                            <div className="flex items-center justify-between gap-2">
                              <div>
                                <div className="flex items-center gap-1.5">
                                  <button
                                    onClick={() => setSelectedAgentProfile(agent)}
                                    className="font-bold text-white hover:text-[#ff5a00] transition cursor-pointer"
                                  >
                                    {agent.name}
                                  </button>
                                  <span className="text-[10px] font-bold px-1.5 py-0.5 rounded bg-[#121316] text-[#ff5a00] border border-[#ff5a00]/40">
                                    {agent.agentCode}
                                  </span>
                                </div>
                                <p className="text-xs font-bold text-[#ff5a00] mt-0.5">{agent.campaign}</p>
                                <p className="text-[11px] text-slate-300">{agent.shift}</p>
                              </div>
                              <button
                                onClick={() => handleMarkAll5DaysPresent(agent.id)}
                                className="text-[10px] font-extrabold px-2 py-1 rounded-lg bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] transition cursor-pointer shrink-0"
                                title="Mark all 5 days Present"
                              >
                                5/5 Present
                              </button>
                            </div>
                          </td>

                          {DAYS.map((day, idx) => {
                            const status = att.days?.[day.key] || 'Present';
                            const agentDayName = CAMPAIGN_CONFIG[agent.campaign]?.days[idx] || day.short;
                            return (
                              <td key={day.key} className="py-3 px-2.5 text-center border-l border-[#454d5e]">
                                <span className="block text-[10px] font-bold text-slate-400 mb-1 uppercase">
                                  {agentDayName}
                                </span>
                                <select
                                  value={status}
                                  onChange={(e) => handleSetDayAttendance(agent.id, day.key, e.target.value)}
                                  className={`w-full text-xs font-extrabold px-2.5 py-2 rounded-xl border text-center cursor-pointer focus:outline-none transition ${getAttendanceStyle(status)}`}
                                >
                                  {ATTENDANCE_STATUSES.map(st => (
                                    <option key={st} value={st} className="bg-[#121316] text-white font-semibold">
                                      {st}
                                    </option>
                                  ))}
                                </select>
                              </td>
                            );
                          })}

                          <td className="py-3.5 px-4 text-center bg-[#252932] border-l-2 border-[#ff5a00]">
                            <div className="flex items-center justify-center gap-1.5">
                              <span className="text-base font-extrabold text-white">
                                {att.scoreDays} / 5 Days
                              </span>
                              <span
                                className={`text-xs font-extrabold px-2 py-0.5 rounded-full ${
                                  att.attPercent >= 90
                                    ? 'bg-[#ff5a00] text-[#121316]'
                                    : att.attPercent >= 75
                                    ? 'bg-amber-500/20 text-amber-300 border border-amber-500/40'
                                    : 'bg-rose-500/20 text-rose-400 border border-rose-500/40'
                                }`}
                              >
                                {att.attPercent}%
                              </span>
                            </div>
                            <div className="text-[11px] text-slate-300 mt-1 flex items-center justify-center gap-2">
                              {att.lateCount > 0 && (
                                <span className="text-amber-300 font-semibold">{att.lateCount} Late</span>
                              )}
                              {att.absentCount > 0 && (
                                <span className="text-rose-400 font-semibold">{att.absentCount} Absent</span>
                              )}
                              {att.lateCount === 0 && att.absentCount === 0 && (
                                <span className="text-[#ff5a00] font-bold">Perfect Week</span>
                              )}
                            </div>
                          </td>

                          <td className="py-3.5 px-4 border-l border-[#454d5e]">
                            <input
                              type="text"
                              value={att.note}
                              onChange={(e) => handleUpdateWeekNote(agent.id, e.target.value)}
                              placeholder="Add weekly attendance remark..."
                              className="w-full px-3 py-1.5 text-xs bg-[#121316] text-white placeholder-slate-400 border border-[#4a5264] rounded-xl focus:outline-none focus:border-[#ff5a00]"
                            />
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* =====================================================================
            TAB 3: FLOOR ANALYTICS, CAMPAIGN BREAKDOWN & COACHING ALERTS
           ===================================================================== */}
        {activeTab === 'analytics' && (
          <div className="space-y-6">
            <div>
              <h2 className="text-base font-extrabold text-white mb-3 flex items-center gap-2">
                <Activity className="w-5 h-5 text-[#ff5a00]" />
                <span>Campaign Performance Comparison — {currentWeekObj.dateRange}</span>
              </h2>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                {analyticsData.campaignStats.map(c => (
                  <div
                    key={c.campaign}
                    className="bg-[#121316] p-5 rounded-2xl border-t-4 border-[#ff5a00] border-l border-r border-b border-[#3f4654] shadow-xl space-y-4"
                  >
                    <div className="flex items-start justify-between">
                      <div>
                        <h3 className="text-base font-black text-white">{c.campaign}</h3>
                        <p className="text-xs text-[#ff5a00] font-bold">{c.shift}</p>
                      </div>
                      <span className="text-xs font-extrabold px-2.5 py-1 rounded-xl bg-[#2b303a] text-slate-200 border border-[#454d5e]">
                        {c.agentCount} Agents
                      </span>
                    </div>

                    <div className="grid grid-cols-3 gap-2 text-center bg-[#252932] p-3 rounded-xl border border-[#3f4654]">
                      <div>
                        <span className="text-[10px] uppercase font-bold text-slate-400 block">Booked</span>
                        <span className="text-xl font-black text-white">{c.booked}</span>
                      </div>
                      <div className="border-x border-[#3f4654]">
                        <span className="text-[10px] uppercase font-bold text-slate-400 block">Shows</span>
                        <span className="text-xl font-black text-[#ff5a00]">{c.shows}</span>
                      </div>
                      <div>
                        <span className="text-[10px] uppercase font-bold text-slate-400 block">Show Rate</span>
                        <span className="text-xl font-black text-white">{c.showRate}%</span>
                      </div>
                    </div>

                    <div className="space-y-1.5">
                      <div className="flex justify-between text-xs font-bold">
                        <span className="text-slate-400">Show Conversion</span>
                        <span className="text-[#ff5a00]">{c.showRate}%</span>
                      </div>
                      <div className="w-full h-2 bg-[#2b303a] rounded-full overflow-hidden">
                        <div
                          className="h-full bg-[#ff5a00] rounded-full transition-all duration-300"
                          style={{ width: `${Math.min(100, c.showRate)}%` }}
                        />
                      </div>
                      <div className="flex justify-between text-xs font-bold pt-1">
                        <span className="text-slate-400">5-Day Attendance</span>
                        <span className="text-emerald-400">{c.attRate}%</span>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>

            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <div className="bg-[#353b48] p-6 rounded-2xl border border-[#4a5264] shadow-xl flex flex-col justify-between">
                <div>
                  <div className="flex items-center justify-between mb-4">
                    <div>
                      <h3 className="text-base font-extrabold text-white">
                        Multi-Week Floor Progression
                      </h3>
                      <p className="text-xs text-slate-300">
                        Comparing Total Booked vs. Confirmed Shows across all recorded 5-day weeks
                      </p>
                    </div>
                    <TrendingUp className="w-5 h-5 text-[#ff5a00]" />
                  </div>

                  <div className="space-y-4 mt-4">
                    {analyticsData.multiWeekTrend.map(wk => {
                      const maxScale = Math.max(150, wk.booked + 10);
                      const bookedWidth = Math.min(100, Math.round((wk.booked / maxScale) * 100));
                      const showsWidth = Math.min(100, Math.round((wk.shows / maxScale) * 100));
                      const isCurrent = wk.id === selectedWeekId;

                      return (
                        <div
                          key={wk.id}
                          onClick={() => setSelectedWeekId(wk.id)}
                          className={`p-3.5 rounded-xl border transition cursor-pointer ${
                            isCurrent
                              ? 'bg-[#121316] border-[#ff5a00]'
                              : 'bg-[#252932] border-[#454d5e] hover:border-slate-400'
                          }`}
                        >
                          <div className="flex items-center justify-between text-xs font-bold mb-2">
                            <span className="text-white flex items-center gap-2">
                              {wk.dateRange}
                              {isCurrent && (
                                <span className="text-[10px] px-2 py-0.5 rounded bg-[#ff5a00] text-[#121316] font-black">
                                  Selected
                                </span>
                              )}
                            </span>
                            <span className="text-[#ff5a00]">
                              {wk.shows} Shows / {wk.booked} Booked ({wk.showRate}%)
                            </span>
                          </div>
                          <div className="space-y-1.5">
                            <div className="flex items-center gap-2">
                              <span className="text-[10px] w-14 text-slate-400 font-bold">Booked</span>
                              <div className="flex-1 h-2.5 bg-[#1c1f26] rounded-full overflow-hidden">
                                <div
                                  className="h-full bg-slate-400 rounded-full"
                                  style={{ width: `${bookedWidth}%` }}
                                />
                              </div>
                            </div>
                            <div className="flex items-center gap-2">
                              <span className="text-[10px] w-14 text-[#ff5a00] font-bold">Shows</span>
                              <div className="flex-1 h-2.5 bg-[#1c1f26] rounded-full overflow-hidden">
                                <div
                                  className="h-full bg-[#ff5a00] rounded-full"
                                  style={{ width: `${showsWidth}%` }}
                                />
                              </div>
                            </div>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                </div>
              </div>

              <div className="bg-[#353b48] p-6 rounded-2xl border border-[#4a5264] shadow-xl">
                <div className="flex items-center justify-between mb-4">
                  <div>
                    <h3 className="text-base font-extrabold text-white flex items-center gap-2">
                      <AlertCircle className="w-5 h-5 text-rose-400" />
                      <span>Supervisor Coaching & Attendance Alerts</span>
                    </h3>
                    <p className="text-xs text-slate-300">
                      Automatically flags agents needing prospect-confirmation coaching or shift follow-up
                    </p>
                  </div>
                </div>

                {analyticsData.alerts.length === 0 ? (
                  <div className="bg-[#121316] p-8 rounded-2xl border border-emerald-500/40 text-center space-y-2">
                    <Check className="w-8 h-8 text-emerald-400 mx-auto" />
                    <p className="text-sm font-extrabold text-white">All Floor Agents On Track!</p>
                    <p className="text-xs text-slate-400">
                      No low show rates or attendance issues detected for {currentWeekObj.dateRange}.
                    </p>
                  </div>
                ) : (
                  <div className="space-y-3">
                    {analyticsData.alerts.map(({ agent, att, reasons }) => (
                      <div
                        key={agent.id}
                        className="p-4 rounded-xl bg-[#121316] border-l-4 border-rose-500 border-t border-r border-b border-[#3f4654] flex flex-col sm:flex-row sm:items-center justify-between gap-3"
                      >
                        <div>
                          <div className="flex items-center gap-2">
                            <button
                              onClick={() => setSelectedAgentProfile(agent)}
                              className="font-extrabold text-white hover:text-[#ff5a00] cursor-pointer"
                            >
                              {agent.name}
                            </button>
                            <span className="text-[10px] font-bold px-1.5 py-0.5 rounded bg-[#2b303a] text-[#ff5a00]">
                              {agent.agentCode}
                            </span>
                            <span className="text-xs text-slate-400">• {agent.campaign}</span>
                          </div>
                          <div className="flex flex-wrap gap-1.5 mt-2">
                            {reasons.map((r, i) => (
                              <span
                                key={i}
                                className="text-[11px] font-bold px-2 py-0.5 rounded-md bg-rose-500/15 text-rose-300 border border-rose-500/30"
                              >
                                {r}
                              </span>
                            ))}
                          </div>
                          {att.note && (
                            <p className="text-[11px] text-slate-400 mt-1.5 italic">
                              Note: "{att.note}"
                            </p>
                          )}
                        </div>
                        <button
                          onClick={() => setSelectedAgentProfile(agent)}
                          className="px-3 py-1.5 rounded-xl bg-[#2b303a] hover:bg-[#ff5a00] hover:text-[#121316] text-slate-200 text-xs font-extrabold transition cursor-pointer shrink-0"
                        >
                          Inspect History
                        </button>
                      </div>
                    ))}
                  </div>
                )}
              </div>
            </div>
          </div>
        )}

        {/* =====================================================================
            TAB 4: SEPARATE MANAGE AGENTS TAB (ADD & REMOVE AGENTS)
           ===================================================================== */}
        {activeTab === 'agents' && (
          <div className="space-y-6">
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <div className="bg-[#121316] p-6 rounded-2xl border-2 border-[#ff5a00] shadow-xl h-fit">
                <div className="flex items-center gap-3 pb-4 border-b border-[#2b303a]">
                  <div className="w-10 h-10 rounded-xl bg-[#ff5a00] text-[#121316] flex items-center justify-center shrink-0">
                    <UserPlus className="w-5 h-5 stroke-[2.5]" />
                  </div>
                  <div>
                    <h2 className="text-base font-extrabold text-white">Add New Floor Agent</h2>
                    <p className="text-xs text-slate-400">
                      Automatically adds agent to 5-Day Performance & Attendance sheets
                    </p>
                  </div>
                </div>

                <form onSubmit={handleCreateAgent} className="space-y-4 mt-4">
                  <div>
                    <label className="text-xs font-bold text-slate-300 block mb-1">
                      Agent Full Name *
                    </label>
                    <input
                      type="text"
                      required
                      value={newAgent.name}
                      onChange={(e) => setNewAgent({ ...newAgent, name: e.target.value })}
                      placeholder="e.g. Shakib Al Hasan"
                      className="w-full px-3.5 py-2.5 bg-[#2b303a] text-white placeholder-slate-400 border border-[#4a5264] rounded-xl text-sm focus:outline-none focus:border-[#ff5a00]"
                    />
                  </div>

                  <div className="grid grid-cols-2 gap-3">
                    <div>
                      <label className="text-xs font-bold text-slate-300 block mb-1">
                        Agent ID Code
                      </label>
                      <input
                        type="text"
                        value={newAgent.agentCode}
                        onChange={(e) => setNewAgent({ ...newAgent, agentCode: e.target.value })}
                        placeholder="DA-108"
                        className="w-full px-3.5 py-2.5 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-sm font-bold focus:outline-none focus:border-[#ff5a00]"
                      />
                    </div>
                    <div>
                      <label className="text-xs font-bold text-slate-300 block mb-1">
                        Phone Number
                      </label>
                      <input
                        type="text"
                        value={newAgent.phone}
                        onChange={(e) => setNewAgent({ ...newAgent, phone: e.target.value })}
                        placeholder="01711-000000"
                        className="w-full px-3.5 py-2.5 bg-[#2b303a] text-white placeholder-slate-400 border border-[#4a5264] rounded-xl text-sm focus:outline-none focus:border-[#ff5a00]"
                      />
                    </div>
                  </div>

                  <div>
                    <label className="text-xs font-bold text-slate-300 block mb-1">
                      Assigned Campaign
                    </label>
                    <select
                      value={newAgent.campaign}
                      onChange={(e) => {
                        const selectedCamp = e.target.value;
                        const autoShift = CAMPAIGN_CONFIG[selectedCamp]?.shift || newAgent.shift;
                        setNewAgent({ ...newAgent, campaign: selectedCamp, shift: autoShift });
                      }}
                      className="w-full px-3.5 py-2.5 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-sm font-semibold focus:outline-none focus:border-[#ff5a00] cursor-pointer"
                    >
                      {CAMPAIGNS.filter(c => c !== 'All Campaigns').map(c => (
                        <option key={c} value={c} className="bg-[#121316] text-white">{c}</option>
                      ))}
                    </select>
                  </div>

                  <div>
                    <label className="text-xs font-bold text-slate-300 block mb-1">
                      Campaign Shift Timing (Auto-Synced)
                    </label>
                    <select
                      value={newAgent.shift}
                      onChange={(e) => setNewAgent({ ...newAgent, shift: e.target.value })}
                      className="w-full px-3.5 py-2.5 bg-[#2b303a] text-[#ff5a00] border border-[#4a5264] rounded-xl text-sm font-bold focus:outline-none focus:border-[#ff5a00] cursor-pointer"
                    >
                      {SHIFTS.map(s => (
                        <option key={s} value={s} className="bg-[#121316] text-white">{s}</option>
                      ))}
                    </select>
                  </div>

                  <button
                    type="submit"
                    className="w-full py-3 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] font-black text-sm shadow-lg shadow-[#ff5a00]/25 transition cursor-pointer flex items-center justify-center gap-2"
                  >
                    <Plus className="w-4 h-4 stroke-[3]" />
                    <span>Add Agent to Floor Roster</span>
                  </button>
                </form>
              </div>

              <div className="lg:col-span-2 bg-[#353b48] rounded-2xl border border-[#4a5264] shadow-xl overflow-hidden">
                <div className="px-6 py-4 border-b border-[#4a5264] bg-[#252932] flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                  <div>
                    <div className="flex items-center gap-2">
                      <h2 className="text-base font-extrabold text-white">
                        Active Dialify Asia Floor Roster ({filteredAgents.length} Agents)
                      </h2>
                      <span className="text-xs font-extrabold px-2.5 py-0.5 rounded-full bg-[#ff5a00] text-[#121316]">
                        Roster Management
                      </span>
                    </div>
                    <p className="text-xs text-slate-300 mt-0.5">
                      Update an agent's campaign or shift anytime, or remove an agent from the floor using the <strong className="text-rose-400">Remove Agent</strong> button.
                    </p>
                  </div>
                </div>

                <div className="overflow-x-auto">
                  <table className="w-full text-left border-collapse">
                    <thead>
                      <tr className="bg-[#121316] border-b-2 border-[#ff5a00] text-[11px] font-extrabold uppercase tracking-wider text-slate-300">
                        <th className="py-3.5 px-4">Agent & ID</th>
                        <th className="py-3.5 px-3 border-l border-[#2b303a]">Campaign</th>
                        <th className="py-3.5 px-3 border-l border-[#2b303a]">Shift</th>
                        <th className="py-3.5 px-3 text-center border-l border-[#2b303a]">Selected Week Stats</th>
                        <th className="py-3.5 px-4 text-right border-l border-[#2b303a]">Action</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-[#454d5e] text-sm">
                      {filteredAgents.length === 0 ? (
                        <tr>
                          <td colSpan={5} className="py-10 text-center text-slate-300">
                            No agents match your current filter. Add a new agent on the left!
                          </td>
                        </tr>
                      ) : (
                        filteredAgents.map(agent => {
                          const perf = getAgentWeekPerf(agent, selectedWeekId);
                          const isConfirming = agentToDelete?.id === agent.id;

                          return (
                            <tr key={agent.id} className="hover:bg-[#3e4554] transition">
                              <td className="py-3.5 px-4">
                                <div className="flex items-center gap-2.5">
                                  <div className="w-9 h-9 rounded-xl bg-[#ff5a00] text-[#121316] font-black text-xs flex items-center justify-center shrink-0">
                                    {String(agent.name || 'A').split(' ').map(n => n[0]).join('')}
                                  </div>
                                  <div>
                                    <div className="flex items-center gap-1.5">
                                      <button
                                        onClick={() => setSelectedAgentProfile(agent)}
                                        className="font-bold text-white hover:text-[#ff5a00] transition cursor-pointer"
                                      >
                                        {agent.name}
                                      </button>
                                      <span className="text-[10px] font-bold px-1.5 py-0.5 rounded bg-[#121316] text-[#ff5a00] border border-[#ff5a00]/40">
                                        {agent.agentCode}
                                      </span>
                                    </div>
                                    <p className="text-xs text-slate-300 mt-0.5">{agent.phone}</p>
                                  </div>
                                </div>
                              </td>

                              <td className="py-3 px-3 border-l border-[#454d5e]">
                                <select
                                  value={agent.campaign}
                                  onChange={(e) => handleUpdateAgentField(agent.id, 'campaign', e.target.value)}
                                  className="px-2.5 py-1.5 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-xs font-semibold focus:outline-none focus:border-[#ff5a00] cursor-pointer"
                                >
                                  {CAMPAIGNS.filter(c => c !== 'All Campaigns').map(c => (
                                    <option key={c} value={c} className="bg-[#121316] text-white">{c}</option>
                                  ))}
                                </select>
                              </td>

                              <td className="py-3 px-3 border-l border-[#454d5e]">
                                <select
                                  value={agent.shift}
                                  onChange={(e) => handleUpdateAgentField(agent.id, 'shift', e.target.value)}
                                  className="px-2.5 py-1.5 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-xs font-semibold focus:outline-none focus:border-[#ff5a00] cursor-pointer"
                                >
                                  {SHIFTS.map(s => (
                                    <option key={s} value={s} className="bg-[#121316] text-white">{s}</option>
                                  ))}
                                </select>
                              </td>

                              <td className="py-3 px-3 text-center border-l border-[#454d5e]">
                                <div className="inline-flex items-center gap-2 px-2.5 py-1 rounded-xl bg-[#252932] border border-[#4a5264] text-xs">
                                  <span className="text-slate-300">
                                    <strong className="text-white">{perf.totalBooked}</strong> Booked
                                  </span>
                                  <span className="text-slate-500">•</span>
                                  <span className="text-[#ff5a00] font-bold">
                                    {perf.totalShows} Shows ({perf.showRate}%)
                                  </span>
                                </div>
                              </td>

                              <td className="py-3 px-4 text-right border-l border-[#454d5e]">
                                {isConfirming ? (
                                  <div className="inline-flex items-center gap-1.5 bg-[#121316] p-1.5 rounded-xl border border-rose-500">
                                    <span className="text-[11px] font-bold text-rose-400 px-1">
                                      Confirm?
                                    </span>
                                    <button
                                      onClick={() => handleConfirmRemoveAgent(agent)}
                                      className="px-2.5 py-1 rounded-lg bg-rose-600 hover:bg-rose-500 text-white text-xs font-extrabold cursor-pointer transition"
                                    >
                                      Yes, Remove
                                    </button>
                                    <button
                                      onClick={() => setAgentToDelete(null)}
                                      className="px-2 py-1 rounded-lg bg-[#2b303a] hover:bg-slate-700 text-slate-300 text-xs font-semibold cursor-pointer transition"
                                    >
                                      Cancel
                                    </button>
                                  </div>
                                ) : (
                                  <button
                                    onClick={() => setAgentToDelete(agent)}
                                    className="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-xl bg-rose-500/15 hover:bg-rose-500/30 text-rose-400 border border-rose-500/40 text-xs font-bold transition cursor-pointer"
                                  >
                                    <Trash2 className="w-3.5 h-3.5" />
                                    <span>Remove Agent</span>
                                  </button>
                                )}
                              </td>
                            </tr>
                          );
                        })
                      )}
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        )}
      </main>

      {/* MODAL: AGENT MULTI-WEEK PROFILE DRILL-DOWN */}
      {selectedAgentProfile && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-xs flex items-center justify-center p-4">
          <div className="bg-[#2b303a] rounded-2xl max-w-2xl w-full p-6 shadow-2xl border-2 border-[#ff5a00] text-white max-h-[90vh] overflow-y-auto">
            <div className="flex items-start justify-between pb-4 border-b border-[#454d5e]">
              <div className="flex items-center gap-3">
                <div className="w-12 h-12 rounded-2xl bg-[#ff5a00] text-[#121316] font-black text-base flex items-center justify-center">
                  {String(selectedAgentProfile.name || 'A').split(' ').map(n => n[0]).join('')}
                </div>
                <div>
                  <div className="flex items-center gap-2">
                    <h2 className="text-lg font-black text-white">{selectedAgentProfile.name}</h2>
                    <span className="text-xs font-bold px-2 py-0.5 rounded bg-[#121316] text-[#ff5a00] border border-[#ff5a00]/40">
                      {selectedAgentProfile.agentCode}
                    </span>
                  </div>
                  <p className="text-xs font-bold text-[#ff5a00]">
                    {selectedAgentProfile.campaign} • {selectedAgentProfile.shift}
                  </p>
                  <p className="text-xs text-slate-300 mt-0.5">Phone: {selectedAgentProfile.phone}</p>
                </div>
              </div>
              <button
                onClick={() => setSelectedAgentProfile(null)}
                className="text-slate-400 hover:text-[#ff5a00] cursor-pointer"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="mt-4 space-y-3">
              <h3 className="text-xs font-extrabold uppercase tracking-wider text-slate-300">
                Multi-Week Performance & Attendance History ({weeks.length} Weeks)
              </h3>
              <div className="overflow-x-auto rounded-xl border border-[#454d5e]">
                <table className="w-full text-left border-collapse text-xs">
                  <thead>
                    <tr className="bg-[#121316] text-slate-300 border-b border-[#454d5e]">
                      <th className="py-2.5 px-3">5-Day Week</th>
                      <th className="py-2.5 px-3 text-center">Booked</th>
                      <th className="py-2.5 px-3 text-center text-[#ff5a00]">Shows</th>
                      <th className="py-2.5 px-3 text-center">Show Rate</th>
                      <th className="py-2.5 px-3 text-center">Attendance</th>
                      <th className="py-2.5 px-3">Supervisor Note</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-[#454d5e] bg-[#252932]">
                    {weeks.map(w => {
                      const p = getAgentWeekPerf(selectedAgentProfile, w.id);
                      const att = getAgentWeekAtt(selectedAgentProfile, w.id);
                      return (
                        <tr key={w.id}>
                          <td className="py-2.5 px-3 font-bold text-white">{w.dateRange}</td>
                          <td className="py-2.5 px-3 text-center font-bold">{p.totalBooked}</td>
                          <td className="py-2.5 px-3 text-center font-black text-[#ff5a00]">{p.totalShows}</td>
                          <td className="py-2.5 px-3 text-center font-bold">{p.showRate}%</td>
                          <td className="py-2.5 px-3 text-center font-bold">
                            {att.scoreDays}/5 ({att.attPercent}%)
                          </td>
                          <td className="py-2.5 px-3 text-slate-300">{att.note || '—'}</td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>

            <div className="flex justify-end pt-4 mt-4 border-t border-[#454d5e]">
              <button
                onClick={() => setSelectedAgentProfile(null)}
                className="px-5 py-2 rounded-xl bg-[#ff5a00] text-[#121316] text-xs font-extrabold hover:bg-[#e04f00] cursor-pointer"
              >
                Close Profile
              </button>
            </div>
          </div>
        </div>
      )}

      {/* MODAL: CONFIRM RESET DEMO DATA */}
      {confirmResetModal && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-xs flex items-center justify-center p-4">
          <div className="bg-[#2b303a] rounded-2xl max-w-md w-full p-6 shadow-2xl border-2 border-rose-500 text-white">
            <div className="flex items-center justify-between pb-3 border-b border-[#454d5e]">
              <h2 className="text-lg font-extrabold text-white">Reset to Factory Demo Data?</h2>
              <button onClick={() => setConfirmResetModal(false)} className="text-slate-400 hover:text-white">
                <X className="w-5 h-5" />
              </button>
            </div>
            <p className="text-xs text-slate-300 mt-3 leading-relaxed">
              This will restore the original 7 Dialify Asia agents and 3 sample weeks (Oct 26 – Nov 13, 2026) and overwrite your current browser changes.
            </p>
            <div className="flex justify-end gap-2 pt-5">
              <button
                onClick={() => setConfirmResetModal(false)}
                className="px-4 py-2 rounded-xl border border-[#4a5264] text-xs font-semibold text-slate-300 hover:bg-[#121316] cursor-pointer"
              >
                Cancel
              </button>
              <button
                onClick={handleResetDemoData}
                className="px-5 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white text-xs font-extrabold cursor-pointer"
              >
                Yes, Reset Demo Data
              </button>
            </div>
          </div>
        </div>
      )}

      {/* MODAL: CREATE NEW 5-DAY WEEK */}
      {showNewWeekModal && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-xs flex items-center justify-center p-4">
          <div className="bg-[#2b303a] rounded-2xl max-w-md w-full p-6 shadow-2xl border-2 border-[#ff5a00] text-white">
            <div className="flex items-center justify-between pb-3 border-b border-[#454d5e]">
              <h2 className="text-lg font-extrabold text-white">Start New 5-Day Workweek</h2>
              <button onClick={() => setShowNewWeekModal(false)} className="text-slate-400 hover:text-[#ff5a00]">
                <X className="w-5 h-5" />
              </button>
            </div>

            <form onSubmit={handleCreateNewWeek} className="space-y-4 mt-4">
              <div>
                <label className="text-xs font-bold text-slate-300 block mb-1">
                  5-Day Week Date Range (Mon – Fri) *
                </label>
                <input
                  type="text"
                  required
                  value={newWeekLabel}
                  onChange={(e) => setNewWeekLabel(e.target.value)}
                  placeholder="e.g. Nov 16 - Nov 20, 2026"
                  className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm focus:outline-none focus:border-[#ff5a00]"
                />
                <p className="text-[11px] text-slate-400 mt-1">
                  This initializes a fresh 5-day performance & attendance sheet for all agents.
                </p>
              </div>

              <div className="flex justify-end gap-2 pt-2">
                <button
                  type="button"
                  onClick={() => setShowNewWeekModal(false)}
                  className="px-4 py-2 rounded-xl border border-[#4a5264] text-xs font-semibold text-slate-300 hover:bg-[#121316]"
                >
                  Cancel
                </button>
                <button
                  type="submit"
                  className="px-5 py-2 rounded-xl bg-[#ff5a00] text-[#121316] text-xs font-extrabold hover:bg-[#e04f00] cursor-pointer"
                >
                  Create 5-Day Week
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* MODAL: QUICK LOG DAY ENTRY */}
      {showQuickLogModal && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-xs flex items-center justify-center p-4">
          <div className="bg-[#2b303a] rounded-2xl max-w-md w-full p-6 shadow-2xl border-2 border-[#ff5a00] text-white">
            <div className="flex items-center justify-between pb-3 border-b border-[#454d5e]">
              <h2 className="text-lg font-extrabold text-white">Log Daily Booked or End-of-Week Shows</h2>
              <button onClick={() => setShowQuickLogModal(false)} className="text-slate-400 hover:text-[#ff5a00]">
                <X className="w-5 h-5" />
              </button>
            </div>

            <form onSubmit={handleQuickLogSubmit} className="space-y-4 mt-4">
              <div>
                <label className="text-xs font-bold text-slate-300 block mb-1">Select Agent</label>
                <select
                  value={quickLog.agentId}
                  onChange={(e) => setQuickLog({ ...quickLog, agentId: e.target.value })}
                  className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm font-semibold"
                >
                  {agents.map(a => (
                    <option key={a.id} value={a.id}>{a.name} ({a.agentCode})</option>
                  ))}
                </select>
              </div>

              <div className="grid grid-cols-2 gap-3">
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">
                    Day for Booked
                  </label>
                  <select
                    value={quickLog.dayKey}
                    onChange={(e) => setQuickLog({ ...quickLog, dayKey: e.target.value })}
                    className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm font-semibold"
                  >
                    {DAYS.map(d => (
                      <option key={d.key} value={d.key}>{d.label}</option>
                    ))}
                  </select>
                </div>
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">Add Daily Booked (+)</label>
                  <input
                    type="number"
                    min="0"
                    value={quickLog.addBooked}
                    onChange={(e) => setQuickLog({ ...quickLog, addBooked: e.target.value })}
                    className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm font-bold"
                  />
                </div>
              </div>

              <div className="p-3.5 rounded-xl bg-[#121316] border border-[#ff5a00]">
                <label className="text-xs font-bold text-[#ff5a00] block mb-1">
                  End-of-Week Confirmed Shows to Add (+) — Optional
                </label>
                <input
                  type="number"
                  min="0"
                  value={quickLog.addWeekShows}
                  onChange={(e) => setQuickLog({ ...quickLog, addWeekShows: e.target.value })}
                  className="w-full px-3.5 py-2 bg-[#2b303a] border border-[#ff5a00]/60 rounded-xl text-sm font-black text-[#ff5a00]"
                />
                <p className="text-[11px] text-slate-300 mt-1">
                  You can also type the agent's total weekly shows directly in the orange "End-of-Week Shows" column on Friday.
                </p>
              </div>

              <div className="flex justify-end gap-2 pt-2">
                <button
                  type="button"
                  onClick={() => setShowQuickLogModal(false)}
                  className="px-4 py-2 rounded-xl border border-[#4a5264] text-xs font-semibold text-slate-300 hover:bg-[#121316]"
                >
                  Cancel
                </button>
                <button
                  type="submit"
                  className="px-5 py-2 rounded-xl bg-[#ff5a00] text-[#121316] text-xs font-extrabold hover:bg-[#e04f00] cursor-pointer"
                >
                  Save Entry
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* MODAL: ADD NEW AGENT */}
      {showAgentModal && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-xs flex items-center justify-center p-4">
          <div className="bg-[#2b303a] rounded-2xl max-w-md w-full p-6 shadow-2xl border-2 border-[#ff5a00] text-white">
            <div className="flex items-center justify-between pb-3 border-b border-[#454d5e]">
              <h2 className="text-lg font-extrabold text-white">Add Dialify Asia Agent</h2>
              <button onClick={() => setShowAgentModal(false)} className="text-slate-400 hover:text-[#ff5a00]">
                <X className="w-5 h-5" />
              </button>
            </div>

            <form onSubmit={handleCreateAgent} className="space-y-4 mt-4">
              <div className="grid grid-cols-3 gap-3">
                <div className="col-span-2">
                  <label className="text-xs font-bold text-slate-300 block mb-1">Agent Full Name *</label>
                  <input
                    type="text"
                    required
                    value={newAgent.name}
                    onChange={(e) => setNewAgent({ ...newAgent, name: e.target.value })}
                    placeholder="e.g. Mahmudul Hasan"
                    className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm"
                  />
                </div>
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">Agent ID</label>
                  <input
                    type="text"
                    value={newAgent.agentCode}
                    onChange={(e) => setNewAgent({ ...newAgent, agentCode: e.target.value })}
                    className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm"
                  />
                </div>
              </div>

              <div className="grid grid-cols-2 gap-3">
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">Campaign</label>
                  <select
                    value={newAgent.campaign}
                    onChange={(e) => {
                      const selectedCamp = e.target.value;
                      const autoShift = CAMPAIGN_CONFIG[selectedCamp]?.shift || newAgent.shift;
                      setNewAgent({ ...newAgent, campaign: selectedCamp, shift: autoShift });
                    }}
                    className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm"
                  >
                    {CAMPAIGNS.filter(c => c !== 'All Campaigns').map(c => (
                      <option key={c} value={c}>{c}</option>
                    ))}
                  </select>
                </div>
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">Shift</label>
                  <select
                    value={newAgent.shift}
                    onChange={(e) => setNewAgent({ ...newAgent, shift: e.target.value })}
                    className="w-full px-3.5 py-2 bg-[#121316] text-white border border-[#4a5264] rounded-xl text-sm"
                  >
                    {SHIFTS.map(s => (
                      <option key={s} value={s}>{s}</option>
                    ))}
                  </select>
                </div>
              </div>

              <div className="flex justify-end gap-2 pt-2">
                <button
                  type="button"
                  onClick={() => setShowAgentModal(false)}
                  className="px-4 py-2 rounded-xl border border-[#4a5264] text-xs font-semibold text-slate-300 hover:bg-[#121316]"
                >
                  Cancel
                </button>
                <button
                  type="submit"
                  className="px-5 py-2 rounded-xl bg-[#ff5a00] text-[#121316] text-xs font-extrabold hover:bg-[#e04f00] cursor-pointer"
                >
                  Add Agent
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* MODAL: SUPER ADMIN SECURITY, CREDENTIALS & AUDIT LOG */}
      {showSecurityModal && isSuperAdmin && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-xs flex items-center justify-center p-4">
          <div className="bg-[#2b303a] rounded-2xl max-w-3xl w-full p-6 shadow-2xl border-2 border-[#ff5a00] text-white max-h-[90vh] overflow-y-auto space-y-6">
            <div className="flex items-center justify-between pb-3 border-b border-[#454d5e]">
              <div className="flex items-center gap-2.5">
                <div className="w-9 h-9 rounded-xl bg-[#ff5a00] text-[#121316] flex items-center justify-center">
                  <Shield className="w-5 h-5 stroke-[2.5]" />
                </div>
                <div>
                  <h2 className="text-lg font-extrabold text-white">
                    Dialify Asia — Admin Security & Audit Control
                  </h2>
                  <p className="text-xs text-slate-300">
                    Manage authorized floor administrators, update passcodes, and review security audit logs
                  </p>
                </div>
              </div>
              <button
                onClick={() => setShowSecurityModal(false)}
                className="text-slate-400 hover:text-[#ff5a00] cursor-pointer"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 gap-5">
              <form
                onSubmit={handleChangeAdminPasscode}
                className="bg-[#121316] p-4 rounded-2xl border border-[#3f4654] space-y-3"
              >
                <h3 className="text-xs font-extrabold uppercase tracking-wider text-[#ff5a00] flex items-center gap-1.5">
                  <KeyRound className="w-4 h-4" />
                  <span>Update Admin Passcode</span>
                </h3>
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">Select Admin Account</label>
                  <select
                    value={passwordChangeForm.targetUsername}
                    onChange={(e) =>
                      setPasswordChangeForm({ ...passwordChangeForm, targetUsername: e.target.value })
                    }
                    className="w-full px-3 py-2 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-xs font-bold"
                  >
                    {adminAccounts.map(a => (
                      <option key={a.id} value={a.username}>
                        @{a.username} — {a.name} ({a.role})
                      </option>
                    ))}
                  </select>
                </div>
                <div>
                  <label className="text-xs font-bold text-slate-300 block mb-1">New Passcode</label>
                  <input
                    type="text"
                    required
                    value={passwordChangeForm.newPasscode}
                    onChange={(e) =>
                      setPasswordChangeForm({ ...passwordChangeForm, newPasscode: e.target.value })
                    }
                    placeholder="Enter new passcode..."
                    className="w-full px-3 py-2 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-xs font-bold"
                  />
                </div>
                <button
                  type="submit"
                  className="w-full py-2 rounded-xl bg-[#ff5a00] hover:bg-[#e04f00] text-[#121316] font-extrabold text-xs cursor-pointer"
                >
                  Save New Passcode
                </button>
              </form>

              <form
                onSubmit={handleCreateNewAdminAccount}
                className="bg-[#121316] p-4 rounded-2xl border border-[#3f4654] space-y-3"
              >
                <h3 className="text-xs font-extrabold uppercase tracking-wider text-[#ff5a00] flex items-center gap-1.5">
                  <UserPlus className="w-4 h-4" />
                  <span>Create Admin / Supervisor Account</span>
                </h3>
                <div className="grid grid-cols-2 gap-2">
                  <div>
                    <label className="text-[11px] font-bold text-slate-300 block mb-1">Username *</label>
                    <input
                      type="text"
                      required
                      value={newAdminForm.username}
                      onChange={(e) => setNewAdminForm({ ...newAdminForm, username: e.target.value })}
                      placeholder="e.g. tl_timeshare"
                      className="w-full px-2.5 py-1.5 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-xs"
                    />
                  </div>
                  <div>
                    <label className="text-[11px] font-bold text-slate-300 block mb-1">Display Name *</label>
                    <input
                      type="text"
                      required
                      value={newAdminForm.name}
                      onChange={(e) => setNewAdminForm({ ...newAdminForm, name: e.target.value })}
                      placeholder="e.g. Tariq (Team Lead)"
                      className="w-full px-2.5 py-1.5 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-xs"
                    />
                  </div>
                </div>
                <div className="grid grid-cols-2 gap-2">
                  <div>
                    <label className="text-[11px] font-bold text-slate-300 block mb-1">Role</label>
                    <select
                      value={newAdminForm.role}
                      onChange={(e) => setNewAdminForm({ ...newAdminForm, role: e.target.value })}
                      className="w-full px-2.5 py-1.5 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-xs font-bold"
                    >
                      <option value="Floor Supervisor">Floor Supervisor</option>
                      <option value="Super Admin">Super Admin</option>
                    </select>
                  </div>
                  <div>
                    <label className="text-[11px] font-bold text-slate-300 block mb-1">Passcode *</label>
                    <input
                      type="text"
                      required
                      value={newAdminForm.passcode}
                      onChange={(e) => setNewAdminForm({ ...newAdminForm, passcode: e.target.value })}
                      placeholder="Passcode"
                      className="w-full px-2.5 py-1.5 bg-[#2b303a] text-white border border-[#4a5264] rounded-xl text-xs"
                    />
                  </div>
                </div>
                <button
                  type="submit"
                  className="w-full py-2 rounded-xl bg-[#2b303a] hover:bg-[#ff5a00] text-[#ff5a00] hover:text-[#121316] border border-[#ff5a00] font-extrabold text-xs transition cursor-pointer"
                >
                  + Add Authorized Account
                </button>
              </form>
            </div>

            <div className="bg-[#121316] p-4 rounded-2xl border border-[#3f4654]">
              <h3 className="text-xs font-extrabold uppercase tracking-wider text-slate-300 mb-2.5">
                Authorized Admin Accounts ({adminAccounts.length})
              </h3>
              <div className="divide-y divide-[#2b303a] text-xs">
                {adminAccounts.map(acc => (
                  <div key={acc.id} className="py-2 flex items-center justify-between">
                    <div>
                      <span className="font-extrabold text-white">{acc.name}</span>
                      <span className="text-slate-400 ml-2">(@{acc.username})</span>
                      <span className="ml-2 text-[10px] font-black px-2 py-0.5 rounded bg-[#ff5a00]/20 text-[#ff5a00]">
                        {acc.role}
                      </span>
                    </div>
                    <div className="flex items-center gap-3">
                      <span className="text-slate-400">
                        Passcode: <strong className="text-white">{acc.passcode}</strong>
                      </span>
                      {acc.username !== 'admin1' && adminAccounts.length > 1 && (
                        <button
                          onClick={() => handleRemoveAdminAccount(acc)}
                          className="text-rose-400 hover:underline font-bold cursor-pointer"
                        >
                          Remove
                        </button>
                      )}
                    </div>
                  </div>
                ))}
              </div>
            </div>

            <div className="bg-[#121316] p-4 rounded-2xl border border-[#3f4654]">
              <h3 className="text-xs font-extrabold uppercase tracking-wider text-slate-300 mb-2.5 flex items-center gap-1.5">
                <Clock className="w-4 h-4 text-[#ff5a00]" />
                <span>Recent Security & Floor Audit Log</span>
              </h3>
              <div className="max-h-40 overflow-y-auto divide-y divide-[#2b303a] text-xs">
                {auditLogs.map(item => (
                  <div key={item.id} className="py-2 flex flex-col sm:flex-row sm:items-center justify-between gap-1">
                    <div>
                      <span className="font-bold text-[#ff5a00]">{item.actor}: </span>
                      <span className="text-slate-200">{item.action}</span>
                    </div>
                    <span className="text-[10px] text-slate-400 shrink-0">{item.timestamp}</span>
                  </div>
                ))}
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
