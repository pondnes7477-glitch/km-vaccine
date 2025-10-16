km-vaccine/
 ┣ src/
 ┣ public/
 ┣ package.json
 ┣ manifest.json  ✅ (รองรับติดตั้งบนโทรศัพท์)
 ┣ service-worker.js ✅ (ทำงานออฟไลน์ได้)
 ┗ README.md
// คำเตือน: ไฟล์นี้เตรียมสำหรับสร้าง ZIP deploy เป็น PWA (KM Vaccine)
// ใช้ภาษาไทยทั้งหมด และพร้อมติดตั้งบนโทรศัพท์ (Add to Home Screen)
// หลังจากดาวน์โหลด ZIP ให้ทำตามขั้นตอนต่อไป:
// 1. แตกไฟล์ ZIP ลงในเครื่อง
// 2. เปิด Terminal และรัน `npm install` ในโฟลเดอร์โปรเจกต์
// 3. รัน `npm run build` เพื่อสร้างไฟล์ build/
// 4. นำโฟลเดอร์ build/ ไป deploy บน Firebase, Vercel หรือ Netlify
// 5. เปิด URL ที่ deploy ในมือถือ แล้วเลือก "Add to Home Screen" เพื่อใช้งานเป็นแอป

// โค้ด React ของ KM Vaccine (ภาษาไทย, PWA-ready)
import React, { useState } from "react";

export default function VaccineScheduler() {
  const [dob, setDob] = useState("");
  const [results, setResults] = useState(null);

  const MILESTONES = [
    { months: 2, label: "2 เดือน", wed: 1 },
    { months: 4, label: "4 เดือน", wed: 1 },
    { months: 6, label: "6 เดือน", wed: 1 },
    { months: 9, label: "9 เดือน", wed: 3 },
    { months: 12, label: "1 ปี (12 เดือน)", wed: 3 },
    { months: 18, label: "1 ปี 6 เดือน (18 เดือน)", wed: 1 },
    { months: 30, label: "2 ปี 6 เดือน (30 เดือน)", wed: 3 },
    { months: 48, label: "4 ปี (48 เดือน)", wed: 1 }
  ];

  function addMonths(date, months) {
    const d = new Date(date.getTime());
    const day = d.getDate();
    d.setMonth(d.getMonth() + months);
    if (d.getDate() !== day) d.setDate(0);
    return d;
  }

  function firstAndThirdWednesdays(year, month) {
    const first = new Date(year, month, 1);
    const offset = (3 - first.getDay() + 7) % 7;
    const firstWed = new Date(year, month, 1 + offset);
    const thirdWed = new Date(year, month, firstWed.getDate() + 14);
    return { firstWed, thirdWed };
  }

  function sameWedNextMonth(date) {
    const y = date.getFullYear();
    const m = date.getMonth();
    const isFirst = date.getDate() <= 14;
    let nextM = m + 1, nextY = y;
    if (nextM > 11) { nextM = 0; nextY++; }
    const { firstWed, thirdWed } = firstAndThirdWednesdays(nextY, nextM);
    return isFirst ? firstWed : thirdWed;
  }

  const formatDate = (d) => d.toLocaleDateString("th-TH", { year: "numeric", month: "short", day: "numeric" });

  function handleCalculate(e) {
    e.preventDefault();
    if (!dob) return;
    const birth = new Date(dob + "T00:00:00");
    const schedule = MILESTONES.map((m) => {
      const target = addMonths(birth, m.months);
      const { firstWed, thirdWed } = firstAndThirdWednesdays(target.getFullYear(), target.getMonth());
      let appointmentDate = m.wed === 1 ? firstWed : thirdWed;
      if (appointmentDate < target) appointmentDate = sameWedNextMonth(appointmentDate);
      return { milestone: m.label, milestoneDate: formatDate(target), appointment: formatDate(appointmentDate), wed: m.wed };
    });
    setResults({ birth: formatDate(birth), schedule });
  }

  return (
    <div className="min-h-screen bg-gradient-to-br from-white to-cyan-50 p-6 flex items-center justify-center">
      <div className="w-full max-w-3xl bg-white rounded-2xl shadow-xl p-6 border border-cyan-100">
        <h1 className="text-2xl font-bold text-cyan-700 mb-4 text-center">KM Vaccine — ระบบคำนวณและนัดฉีดวัคซีนเด็ก</h1>
        <p className="text-sm text-gray-600 text-center mb-6">บริการเฉพาะพุธที่ 1 และ พุธที่ 3 ของทุกเดือน ตามช่วงอายุวัคซีน</p>

        <form onSubmit={handleCalculate} className="flex flex-col sm:flex-row gap-4 mb-6">
          <label className="flex-1">
            <div className="text-sm text-gray-700 mb-1">วัน/เดือน/ปีเกิดของเด็ก</div>
            <input type="date" value={dob} onChange={(e) => setDob(e.target.value)} className="w-full rounded-lg border p-2" required />
          </label>
          <button className="bg-cyan-600 hover:bg-cyan-700 text-white font-semibold px-4 py-2 rounded-lg">คำนวณ & สร้างนัด</button>
        </form>

        {results && (
          <section className="bg-cyan-50 rounded-xl p-4 shadow-inner">
            <h2 className="text-lg font-semibold text-green-700 mb-2">ผลการคำนวณ</h2>
            <p className="text-sm text-gray-600 mb-3">วันเกิด: {results.birth}</p>

            {results.schedule.map((s) => (
              <div key={s.milestone} className="border rounded-md p-3 mb-2 flex justify-between items-center bg-white">
                <div>
                  <div className="font-medium text-gray-800">{s.milestone}</div>
                  <div className="text-xs text-gray-500">กำหนดอายุ: {s.milestoneDate}</div>
                </div>
                <div className="text-right">
                  <div className="text-sm font-semibold text-cyan-800">นัด: {s.appointment}</div>
                  <div className="text-xs text-gray-500">(พุธที่ {s.wed})</div>
                </div>
              </div>
            ))}
          </section>
        )}

        <footer className="mt-6 text-xs text-gray-500 text-center">ธีมสีฟ้า-เขียว-ขาว • รองรับ PWA ติดตั้งบนมือถือได้</footer>
      </div>
    </div>
  );
}

