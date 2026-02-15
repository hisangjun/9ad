# 9ad
import React, { useState, useMemo } from 'react';
import { 
  Users, 
  Calendar, 
  PlusCircle, 
  History, 
  UserPlus, 
  Search,
  Trash2,
  TrendingUp,
  Info,
  Clock,
  Briefcase,
  Lock,
  Unlock,
  X,
  ShieldCheck,
  Settings,
  KeyRound
} from 'lucide-react';

const App = () => {
  // 설정된 부서 목록
  const DEPARTMENTS = ['영업팀', '마케팅팀', '해외수출팀', '인사팀', '개발팀', '디자인팀'];
  const VACATION_TYPES = ['연차', '반차', '휴가'];

  // 관리자 모드 및 로그인 상태
  const [isAdmin, setIsAdmin] = useState(false);
  const [showAdminModal, setShowAdminModal] = useState(false);
  const [password, setPassword] = useState('');
  const [loginError, setLoginError] = useState('');
  
  // 비밀번호 변경 관련 상태
  const [adminPassword, setAdminPassword] = useState('1234'); // 초기 비밀번호
  const [showChangePwModal, setShowChangePwModal] = useState(false);
  const [newPassword, setNewPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [changePwError, setChangePwError] = useState('');

  // 회계연도 기준 연차 계산 함수
  const calculateLegalLeave = (joinDateStr) => {
    if (!joinDateStr) return 0;
    
    const joinDate = new Date(joinDateStr);
    const today = new Date();
    const currentYear = today.getFullYear();
    const joinYear = joinDate.getFullYear();
    const joinMonth = joinDate.getMonth() + 1;

    const yearCount = currentYear - joinYear + 1;

    if (yearCount === 1) {
      const monthsWorked = today.getMonth() - joinDate.getMonth();
      return Math.max(0, Math.min(monthsWorked, 11));
    } else if (yearCount === 2) {
      const monthsInFirstYear = 12 - joinMonth + 1;
      const proRatedLeave = (15 * monthsInFirstYear) / 12;
      return parseFloat(proRatedLeave.toFixed(1));
    } else {
      const seniorityYears = currentYear - joinYear - 1;
      const additionalDays = Math.floor(seniorityYears / 2);
      return Math.min(15 + additionalDays, 25);
    }
  };

  const [employees, setEmployees] = useState([
    { id: 1, name: '김철수', department: '개발팀', joinDate: '2023-01-01' },
    { id: 2, name: '이영희', department: '마케팅팀', joinDate: '2025-05-15' },
    { id: 3, name: '박지성', department: '영업팀', joinDate: '2020-03-10' },
    { id: 4, name: '최강희', department: '해외수출팀', joinDate: '2022-07-20' },
  ]);

  const [usageLogs, setUsageLogs] = useState([
    { id: 1, employeeId: 1, date: '2025-02-01', type: '연차', days: 1, reason: '개인사유' },
    { id: 2, employeeId: 3, date: '2025-01-20', type: '휴가', days: 3, reason: '리프레시 휴가' },
  ]);

  const [searchTerm, setSearchTerm] = useState('');
  const [activeTab, setActiveTab] = useState('dashboard');

  const employeeStats = useMemo(() => {
    return employees.map(emp => {
      const yearCount = new Date().getFullYear() - new Date(emp.joinDate).getFullYear() + 1;
      const totalLeave = calculateLegalLeave(emp.joinDate);
      const used = usageLogs
        .filter(log => log.employeeId === emp.id)
        .reduce((sum, log) => sum + log.days, 0);
      return { 
        ...emp, 
        yearCount,
        totalLeave, 
        used, 
        remaining: parseFloat((totalLeave - used).toFixed(1))
      };
    });
  }, [employees, usageLogs]);

  const filteredStats = employeeStats.filter(emp => 
    emp.name.includes(searchTerm) || emp.department.includes(searchTerm)
  );

  const [newEmp, setNewEmp] = useState({ name: '', department: '영업팀', joinDate: '' });
  const handleAddEmployee = (e) => {
    e.preventDefault();
    if (!newEmp.name || !newEmp.joinDate) return;
    setEmployees([...employees, { ...newEmp, id: Date.now() }]);
    setNewEmp({ name: '', department: '영업팀', joinDate: '' });
  };

  const [newLog, setNewLog] = useState({ employeeId: '', date: '', type: '연차', days: 1, reason: '' });
  
  // 유형 선택 시 기본 일수 설정 함수
  const handleTypeChange = (type) => {
    let defaultDays = 1;
    if (type === '반차') defaultDays = 0.5;
    if (type === '휴가') defaultDays = 1;
    setNewLog({ ...newLog, type, days: defaultDays });
  };

  const handleAddLog = (e) => {
    e.preventDefault();
    if (!newLog.employeeId || !newLog.date) return;
    setUsageLogs([...usageLogs, { 
      ...newLog, 
      id: Date.now(), 
      employeeId: parseInt(newLog.employeeId), 
      days: parseFloat(newLog.days) 
    }]);
    setNewLog({ employeeId: '', date: '', type: '연차', days: 1, reason: '' });
  };

  const handleDeleteLog = (id) => {
    if (!isAdmin) return;
    setUsageLogs(usageLogs.filter(log => log.id !== id));
  };

  const handleDeleteEmployee = (id) => {
    if (!isAdmin) return;
    setEmployees(employees.filter(emp => emp.id !== id));
    setUsageLogs(usageLogs.filter(log => log.employeeId !== id));
  };

  // 관리자 모드 토글 및 로그인 처리
  const handleAdminToggle = () => {
    if (isAdmin) {
      setIsAdmin(false); // 이미 관리자라면 로그아웃
    } else {
      setShowAdminModal(true); // 관리자가 아니라면 로그인 모달 열기
      setPassword('');
      setLoginError('');
    }
  };

  const handleAdminLogin = (e) => {
    e.preventDefault();
    if (password === adminPassword) {
      setIsAdmin(true);
      setShowAdminModal(false);
    } else {
      setLoginError('비밀번호가 올바르지 않습니다.');
    }
  };

  // 비밀번호 변경 처리
  const handleChangePassword = (e) => {
    e.preventDefault();
    if (newPassword.length < 4) {
      setChangePwError('비밀번호는 4자리 이상이어야 합니다.');
      return;
    }
    if (newPassword !== confirmPassword) {
      setChangePwError('비밀번호가 서로 일치하지 않습니다.');
      return;
    }
    
    setAdminPassword(newPassword);
    setShowChangePwModal(false);
    setNewPassword('');
    setConfirmPassword('');
    setChangePwError('');
    // 성공 피드백 (간단한 alert 사용)
    window.alert('관리자 비밀번호가 성공적으로 변경되었습니다.');
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 font-sans relative">
      {/* 관리자 로그인 모달 */}
      {showAdminModal && (
        <div className="fixed inset-0 bg-black/50 z-50 flex items-center justify-center backdrop-blur-sm p-4">
          <div className="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-sm relative animate-in fade-in zoom-in duration-200">
            <button 
              onClick={() => setShowAdminModal(false)}
              className="absolute top-4 right-4 text-slate-400 hover:text-slate-600"
            >
              <X size={20} />
            </button>
            
            <div className="text-center mb-6">
              <div className="bg-slate-100 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
                <ShieldCheck size={32} className="text-slate-600" />
              </div>
              <h3 className="text-xl font-bold text-slate-800">관리자 인증</h3>
              <p className="text-sm text-slate-500 mt-1">데이터 삭제 권한을 위해 비밀번호를 입력하세요.</p>
            </div>

            <form onSubmit={handleAdminLogin} className="space-y-4">
              <div>
                <input 
                  type="password" 
                  placeholder="비밀번호 입력" 
                  className={`w-full p-3 bg-slate-50 border rounded-xl outline-none focus:ring-2 transition-all text-center font-mono text-lg ${loginError ? 'border-red-300 focus:ring-red-200' : 'border-slate-200 focus:ring-slate-200'}`}
                  value={password}
                  onChange={(e) => setPassword(e.target.value)}
                  autoFocus
                />
                {loginError && <p className="text-xs text-red-500 mt-2 text-center font-medium">{loginError}</p>}
              </div>
              <button 
                type="submit" 
                className="w-full bg-slate-900 text-white font-bold py-3 rounded-xl hover:bg-slate-800 transition-colors"
              >
                인증하기
              </button>
            </form>
          </div>
        </div>
      )}

      {/* 비밀번호 변경 모달 */}
      {showChangePwModal && (
        <div className="fixed inset-0 bg-black/50 z-50 flex items-center justify-center backdrop-blur-sm p-4">
          <div className="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-sm relative animate-in fade-in zoom-in duration-200">
            <button 
              onClick={() => setShowChangePwModal(false)}
              className="absolute top-4 right-4 text-slate-400 hover:text-slate-600"
            >
              <X size={20} />
            </button>
            
            <div className="text-center mb-6">
              <div className="bg-slate-100 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
                <KeyRound size={32} className="text-slate-600" />
              </div>
              <h3 className="text-xl font-bold text-slate-800">비밀번호 변경</h3>
              <p className="text-sm text-slate-500 mt-1">새로운 관리자 비밀번호를 설정합니다.</p>
            </div>

            <form onSubmit={handleChangePassword} className="space-y-4">
              <div className="space-y-2">
                <input 
                  type="password" 
                  placeholder="새 비밀번호" 
                  className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-slate-200 transition-all font-mono text-center"
                  value={newPassword}
                  onChange={(e) => setNewPassword(e.target.value)}
                  required
                />
                <input 
                  type="password" 
                  placeholder="새 비밀번호 확인" 
                  className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-slate-200 transition-all font-mono text-center"
                  value={confirmPassword}
                  onChange={(e) => setConfirmPassword(e.target.value)}
                  required
                />
                {changePwError && <p className="text-xs text-red-500 text-center font-medium">{changePwError}</p>}
              </div>
              <button 
                type="submit" 
                className="w-full bg-blue-600 text-white font-bold py-3 rounded-xl hover:bg-blue-700 transition-colors"
              >
                변경 완료
              </button>
            </form>
          </div>
        </div>
      )}

      <nav className="fixed left-0 top-0 h-full w-64 bg-white border-r border-slate-200 p-6 flex flex-col gap-8 shadow-sm z-10">
        <div className="flex items-center gap-2 text-blue-600 mb-4">
          <Calendar size={32} strokeWidth={2.5} />
          <h1 className="text-xl font-bold tracking-tight">연차관리 Pro</h1>
        </div>
        
        <div className="flex flex-col gap-2">
          <button onClick={() => setActiveTab('dashboard')} className={`flex items-center gap-3 px-4 py-3 rounded-xl transition-all ${activeTab === 'dashboard' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}`}><TrendingUp size={20} /> 대시보드</button>
          <button onClick={() => setActiveTab('employees')} className={`flex items-center gap-3 px-4 py-3 rounded-xl transition-all ${activeTab === 'employees' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}`}><Users size={20} /> 직원 목록</button>
          <button onClick={() => setActiveTab('logs')} className={`flex items-center gap-3 px-4 py-3 rounded-xl transition-all ${activeTab === 'logs' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 hover:bg-slate-100'}`}><History size={20} /> 연차 기록</button>
        </div>

        <div className="mt-auto flex flex-col gap-4">
          <div className="flex gap-2">
            <button 
              onClick={handleAdminToggle}
              className={`flex-1 flex items-center justify-between px-4 py-3 rounded-xl transition-all border text-sm font-bold ${isAdmin ? 'bg-slate-800 text-white border-slate-800 shadow-lg' : 'bg-white text-slate-500 border-slate-200 hover:bg-slate-50'}`}
            >
              <div className="flex items-center gap-2">
                {isAdmin ? <Unlock size={18} /> : <Lock size={18} />}
                {isAdmin ? '관리자 ON' : '관리자 OFF'}
              </div>
              <div className={`w-2 h-2 rounded-full ${isAdmin ? 'bg-green-400 animate-pulse' : 'bg-slate-300'}`} />
            </button>
            {isAdmin && (
              <button 
                onClick={() => {
                  setShowChangePwModal(true);
                  setChangePwError('');
                  setNewPassword('');
                  setConfirmPassword('');
                }}
                className="px-3 py-3 rounded-xl bg-white border border-slate-200 text-slate-500 hover:bg-slate-50 hover:text-slate-800 transition-colors"
                title="비밀번호 변경"
              >
                <Settings size={18} />
              </button>
            )}
          </div>

          <div className="p-4 bg-blue-50 rounded-2xl border border-blue-100">
            <div className="flex items-center gap-2 text-blue-700 mb-2 font-bold text-sm">
              <Info size={16} /> 연차 유형 가이드
            </div>
            <ul className="text-[11px] text-blue-600 space-y-1">
              <li>• 연차: 기본 1일 차감</li>
              <li>• 반차: 기본 0.5일 차감</li>
              <li>• 휴가: 일수를 직접 입력 가능</li>
              <li>• 회계연도 기준 자동 산정 적용 중</li>
            </ul>
          </div>
        </div>
      </nav>

      <main className="ml-64 p-10">
        <header className="flex justify-between items-center mb-10">
          <div>
            <h2 className="text-3xl font-bold text-slate-800">
              {activeTab === 'dashboard' && "회계연도 현황"}
              {activeTab === 'employees' && "부서별 직원 관리"}
              {activeTab === 'logs' && "사용 내역 및 일수 관리"}
            </h2>
            <p className="text-slate-500 mt-1">부서별 목록과 연차 유형을 자유롭게 관리하세요.</p>
          </div>
          <div className="relative">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" size={18} />
            <input 
              type="text" 
              placeholder="이름 또는 부서 검색..." 
              className="pl-10 pr-4 py-2 bg-white border border-slate-200 rounded-full w-64 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all shadow-sm"
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
            />
          </div>
        </header>

        {activeTab === 'dashboard' && (
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            <div className="bg-white p-6 rounded-3xl border border-slate-100 shadow-sm">
              <span className="text-slate-500 font-medium">관리 대상 직원</span>
              <div className="text-4xl font-black text-blue-600 mt-1">{employees.length}명</div>
            </div>
            <div className="bg-white p-6 rounded-3xl border border-slate-100 shadow-sm">
              <span className="text-slate-500 font-medium">부서 수</span>
              <div className="text-4xl font-black text-emerald-600 mt-1">{new Set(employees.map(e => e.department)).size}개</div>
            </div>
            <div className="bg-white p-6 rounded-3xl border border-slate-100 shadow-sm">
              <span className="text-slate-500 font-medium">전체 잔여 연차</span>
              <div className="text-4xl font-black text-amber-500 mt-1">
                {(filteredStats.reduce((s, e) => s + e.remaining, 0)).toFixed(1)}일
              </div>
            </div>

            <div className="col-span-3 bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden mt-4">
              <div className="p-6 border-b border-slate-50 font-bold text-lg flex items-center gap-2">
                <Clock className="text-blue-500" size={20} />
                2026년도 부서별 연차 현황
              </div>
              <table className="w-full text-left">
                <thead className="bg-slate-50 text-slate-500 text-sm">
                  <tr>
                    <th className="px-6 py-4">이름</th>
                    <th className="px-6 py-4">부서 / 연차</th>
                    <th className="px-6 py-4 text-center">발생 연차</th>
                    <th className="px-6 py-4 text-center">사용</th>
                    <th className="px-6 py-4 text-center">잔여</th>
                    <th className="px-6 py-4">상태</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-slate-50">
                  {filteredStats.map(emp => (
                    <tr key={emp.id} className="hover:bg-slate-50/50 transition-colors">
                      <td className="px-6 py-4 font-bold">{emp.name}</td>
                      <td className="px-6 py-4">
                        <div className="text-sm font-medium text-slate-700">{emp.department}</div>
                        <div className="text-[10px] text-blue-500 font-bold uppercase tracking-wider">{emp.yearCount}년차</div>
                      </td>
                      <td className="px-6 py-4 text-center font-semibold text-slate-700">{emp.totalLeave}</td>
                      <td className="px-6 py-4 text-center text-red-500 font-medium">{emp.used}</td>
                      <td className="px-6 py-4 text-center text-blue-600 font-bold">{emp.remaining}</td>
                      <td className="px-6 py-4">
                        <div className="w-24 bg-slate-100 h-1.5 rounded-full overflow-hidden">
                           <div className="bg-blue-500 h-full" style={{ width: `${Math.min((emp.used/emp.totalLeave)*100, 100)}%` }} />
                        </div>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        )}

        {activeTab === 'employees' && (
          <div className="grid grid-cols-3 gap-8">
            <div className="col-span-2 bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
              <table className="w-full text-left">
                <thead className="bg-slate-50 text-slate-500 text-sm font-semibold">
                  <tr>
                    <th className="px-6 py-4">직원명</th>
                    <th className="px-6 py-4">소속 부서</th>
                    <th className="px-6 py-4">입사일</th>
                    {isAdmin && <th className="px-6 py-4 text-center">관리 (삭제)</th>}
                  </tr>
                </thead>
                <tbody className="divide-y divide-slate-50">
                  {employees.map(emp => (
                    <tr key={emp.id}>
                      <td className="px-6 py-4 font-bold">{emp.name}</td>
                      <td className="px-6 py-4">
                        <span className="bg-slate-100 text-slate-600 px-2 py-1 rounded text-xs">
                          {emp.department}
                        </span>
                      </td>
                      <td className="px-6 py-4 text-slate-600 font-mono text-sm">{emp.joinDate}</td>
                      {isAdmin && (
                        <td className="px-6 py-4 text-center">
                          <button onClick={() => handleDeleteEmployee(emp.id)} className="text-slate-300 hover:text-red-500 p-2 transition-colors">
                            <Trash2 size={18} />
                          </button>
                        </td>
                      )}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
            
            <div className="bg-white p-8 rounded-3xl border border-slate-100 shadow-sm flex flex-col gap-6">
              <div className="flex items-center gap-2 text-blue-600 font-bold text-lg border-b pb-4">
                <UserPlus size={22} />
                <h3>신규 직원 등록</h3>
              </div>
              <form onSubmit={handleAddEmployee} className="flex flex-col gap-5">
                <div className="space-y-1">
                  <label className="text-xs font-bold text-slate-500 ml-1 uppercase">성명</label>
                  <input type="text" className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none" value={newEmp.name} onChange={(e) => setNewEmp({...newEmp, name: e.target.value})} placeholder="이름 입력" required />
                </div>
                <div className="space-y-1">
                  <label className="text-xs font-bold text-slate-500 ml-1 uppercase">부서 선택</label>
                  <select className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-blue-500" value={newEmp.department} onChange={(e) => setNewEmp({...newEmp, department: e.target.value})}>
                    {DEPARTMENTS.map(dept => <option key={dept} value={dept}>{dept}</option>)}
                  </select>
                </div>
                <div className="space-y-1">
                  <label className="text-xs font-bold text-slate-500 ml-1 uppercase">입사일</label>
                  <input type="date" className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-blue-500" value={newEmp.joinDate} onChange={(e) => setNewEmp({...newEmp, joinDate: e.target.value})} required />
                </div>
                <button type="submit" className="mt-2 bg-blue-600 text-white font-bold py-4 rounded-2xl hover:bg-blue-700 transition-all shadow-lg">
                  직원 추가하기
                </button>
              </form>
            </div>
          </div>
        )}

        {activeTab === 'logs' && (
          <div className="grid grid-cols-3 gap-8">
            <div className="col-span-2 bg-white rounded-3xl border border-slate-100 shadow-sm overflow-hidden">
               <table className="w-full text-left">
                <thead className="bg-slate-50 text-slate-500 text-sm">
                  <tr>
                    <th className="px-6 py-4">사용일자</th>
                    <th className="px-6 py-4">이름</th>
                    <th className="px-6 py-4 text-center">유형</th>
                    <th className="px-6 py-4 text-center">일수</th>
                    <th className="px-6 py-4">사유</th>
                    {isAdmin && <th className="px-6 py-4 text-center">삭제</th>}
                  </tr>
                </thead>
                <tbody className="divide-y divide-slate-50">
                  {usageLogs.sort((a,b) => new Date(b.date) - new Date(a.date)).map(log => {
                    const emp = employees.find(e => e.id === log.employeeId);
                    return (
                      <tr key={log.id} className="group">
                        <td className="px-6 py-4 text-slate-600 font-medium">{log.date}</td>
                        <td className="px-6 py-4 font-bold">{emp ? emp.name : '퇴사자'}</td>
                        <td className="px-6 py-4 text-center">
                          <span className={`px-2 py-1 rounded-lg text-[10px] font-black uppercase ${log.type === '연차' ? 'bg-blue-50 text-blue-600' : log.type === '반차' ? 'bg-amber-50 text-amber-600' : 'bg-emerald-50 text-emerald-600'}`}>
                            {log.type}
                          </span>
                        </td>
                        <td className="px-6 py-4 text-center font-bold text-slate-800">{log.days}일</td>
                        <td className="px-6 py-4 text-slate-400 text-sm italic">{log.reason || '-'}</td>
                        {isAdmin && (
                          <td className="px-6 py-4 text-center">
                            <button onClick={() => handleDeleteLog(log.id)} className="text-slate-300 hover:text-red-500 transition-colors p-2">
                              <Trash2 size={16} />
                            </button>
                          </td>
                        )}
                      </tr>
                    )
                  })}
                </tbody>
              </table>
            </div>

            <div className="bg-white p-8 rounded-3xl border border-slate-100 shadow-sm flex flex-col gap-6">
              <div className="flex items-center gap-2 text-emerald-600 font-bold text-lg border-b pb-4">
                <PlusCircle size={22} />
                <h3>연차/휴가 사용 등록</h3>
              </div>
              <form onSubmit={handleAddLog} className="flex flex-col gap-4">
                <div className="space-y-1">
                  <label className="text-xs font-bold text-slate-500 ml-1 uppercase">대상 직원</label>
                  <select className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-emerald-500" value={newLog.employeeId} onChange={(e) => setNewLog({...newLog, employeeId: e.target.value})} required>
                    <option value="">직원 선택</option>
                    {employees.map(e => <option key={e.id} value={e.id}>{e.name} ({e.department})</option>)}
                  </select>
                </div>
                <div className="space-y-1">
                  <label className="text-xs font-bold text-slate-500 ml-1 uppercase">사용일</label>
                  <input type="date" className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-emerald-500" value={newLog.date} onChange={(e) => setNewLog({...newLog, date: e.target.value})} required />
                </div>
                <div className="flex gap-4">
                  <div className="flex-1 space-y-1">
                    <label className="text-xs font-bold text-slate-500 ml-1 uppercase">유형</label>
                    <select className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-emerald-500" value={newLog.type} onChange={(e) => handleTypeChange(e.target.value)}>
                      {VACATION_TYPES.map(type => <option key={type} value={type}>{type}</option>)}
                    </select>
                  </div>
                  <div className="w-1/3 space-y-1">
                    <label className="text-xs font-bold text-slate-500 ml-1 uppercase">일수</label>
                    <input 
                      type="number" step="0.5" min="0.5"
                      className="w-full p-3 bg-white border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-emerald-500 font-bold text-center" 
                      value={newLog.days} 
                      onChange={(e) => setNewLog({...newLog, days: parseFloat(e.target.value)})}
                    />
                  </div>
                </div>
                <div className="space-y-1">
                  <label className="text-xs font-bold text-slate-500 ml-1 uppercase">상세 사유</label>
                  <input type="text" className="w-full p-3 bg-slate-50 border border-slate-200 rounded-xl outline-none focus:ring-2 focus:ring-emerald-500" value={newLog.reason} onChange={(e) => setNewLog({...newLog, reason: e.target.value})} placeholder="사유를 입력하세요" />
                </div>
                <button type="submit" className="mt-2 bg-emerald-600 text-white font-bold py-4 rounded-2xl hover:bg-emerald-700 transition-all shadow-lg active:scale-95">
                  사용 기록 저장
                </button>
              </form>
            </div>
          </div>
        )}
      </main>
    </div>
  );
};

export default App;
