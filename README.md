# Yournal-app
The one stop solution to all your journalling worries!
import Dashboard from "./Dashboard";

function App() {
  return (
    <div>
      <Dashboard />
    </div>
  );
}

export default App;
import React, { useState } from "react";
import DashboardOverview from "./DashboardOverview";
import FocusTimer from "./FocusTimer";
import Journal from "./Journal";
import MoodTracker from "./MoodTracker";
import HabitTracker from "./HabitTracker";
import styles from "./styles/Dashboard.module.css";

export default function Dashboard() {
  // Centralized state for overview
  const [focusMinutes, setFocusMinutes] = useState(0);
  const [journal, setJournal] = useState({
    checklist: [],
    gratitude: "",
    ratings: { work: 0, personal: 0, selfCare: 0 }
  });
  const [mood, setMood] = useState(null); // excited, happy, neutral, sad, angry
  const [habits, setHabits] = useState([
    // { name: "Drink water", streak: 3, doneToday: false }
  ]);

  return (
    <div className={styles.dashboard}>
      <DashboardOverview
        focusMinutes={focusMinutes}
        journal={journal}
        mood={mood}
        habits={habits}
      />
      <div className={styles.sections}>
        <FocusTimer setFocusMinutes={setFocusMinutes} />
        <Journal journal={journal} setJournal={setJournal} />
        <MoodTracker mood={mood} setMood={setMood} />
        <HabitTracker habits={habits} setHabits={setHabits} />
      </div>
    </div>
  );
}
import React from "react";
import styles from "./styles/DashboardOverview.module.css";

const moodIcons = {
  excited: "🤩",
  happy: "😊",
  neutral: "😐",
  sad: "😢",
  angry: "😡"
};

export default function DashboardOverview({ focusMinutes, journal, mood, habits }) {
  const completedHabits = habits.filter(h => h.doneToday).length;
  return (
    <div className={styles.overview}>
      <h2>Today's Overview</h2>
      <div>
        <b>Focus:</b> {focusMinutes} min
      </div>
      <div>
        <b>Today's Mood:</b> {mood ? moodIcons[mood] : "Not set"}
      </div>
      <div>
        <b>Habits Completed:</b> {completedHabits}/{habits.length}
      </div>
    </div>
  );
}
import React, { useState, useRef } from "react";
import styles from "./styles/FocusTimer.module.css";

const DURATIONS = [10, 15, 30, 60];

export default function FocusTimer({ setFocusMinutes }) {
  const [selected, setSelected] = useState(15);
  const [remaining, setRemaining] = useState(0);
  const [active, setActive] = useState(false);
  const timerRef = useRef();

  function startTimer() {
    setRemaining(selected * 60);
    setActive(true);
    timerRef.current = setInterval(() => {
      setRemaining(prev => {
        if (prev <= 1) {
          clearInterval(timerRef.current);
          setActive(false);
          setFocusMinutes(m => m + selected);
          return 0;
        }
        return prev - 1;
      });
    }, 1000);
  }
  function stopTimer() {
    clearInterval(timerRef.current);
    setActive(false);
    setRemaining(0);
  }

  return (
    <div className={styles.timer}>
      <div className={styles.icon}>Y</div>
      <div className={styles.durations}>
        {DURATIONS.map(d => (
          <button
            key={d}
            disabled={active}
            className={selected === d ? styles.selected : ""}
            onClick={() => setSelected(d)}
          >
            {d} min
          </button>
        ))}
      </div>
      <div className={styles.display}>
        {active
          ? `${Math.floor(remaining / 60)
              .toString()
              .padStart(2, "0")}:${(remaining % 60).toString().padStart(2, "0")}`
          : `${selected}:00`}
      </div>
      <div className={styles.controls}>
        {!active ? (
          <button onClick={startTimer}>Start</button>
        ) : (
          <button onClick={stopTimer}>Stop</button>
        )}
      </div>
    </div>
  );
}
import React from "react";
import styles from "./styles/Journal.module.css";

function StarRating({ value, onChange }) {
  return (
    <span>
      {[1, 2, 3, 4, 5].map(n => (
        <span
          key={n}
          className={value >= n ? styles.starActive : styles.star}
          onClick={() => onChange(n)}
          role="button"
        >
          ★
        </span>
      ))}
    </span>
  );
}

export default function Journal({ journal, setJournal }) {
  const handleChecklistChange = idx => {
    const updated = journal.checklist.map((item, i) =>
      i === idx ? { ...item, done: !item.done } : item
    );
    setJournal({ ...journal, checklist: updated });
  };
  const handleAddChecklist = () => {
    setJournal({
      ...journal,
      checklist: [...journal.checklist, { text: "", done: false }]
    });
  };
  const handleChecklistText = (idx, text) => {
    const updated = journal.checklist.map((item, i) =>
      i === idx ? { ...item, text } : item
    );
    setJournal({ ...journal, checklist: updated });
  };

  return (
    <div className={styles.journal}>
      <h3>Journal</h3>
      <div>
        <b>Checklist:</b>
        <ul>
          {journal.checklist.map((item, idx) => (
            <li key={idx}>
              <input
                type="checkbox"
                checked={item.done}
                onChange={() => handleChecklistChange(idx)}
              />
              <input
                type="text"
                value={item.text}
                placeholder="Task..."
                onChange={e => handleChecklistText(idx, e.target.value)}
              />
            </li>
          ))}
        </ul>
        <button onClick={handleAddChecklist}>Add Item</button>
      </div>
      <div>
        <b>Gratitude:</b>
        <textarea
          value={journal.gratitude}
          placeholder="What are you grateful for today?"
          onChange={e => setJournal({ ...journal, gratitude: e.target.value })}
        />
      </div>
      <div>
        <b>Work:</b>
        <StarRating
          value={journal.ratings.work}
          onChange={n =>
            setJournal({ ...journal, ratings: { ...journal.ratings, work: n } })
          }
        />
        <b>Personal:</b>
        <StarRating
          value={journal.ratings.personal}
          onChange={n =>
            setJournal({
              ...journal,
              ratings: { ...journal.ratings, personal: n }
            })
          }
        />
        <b>Self Care:</b>
        <StarRating
          value={journal.ratings.selfCare}
          onChange={n =>
            setJournal({
              ...journal,
              ratings: { ...journal.ratings, selfCare: n }
            })
          }
        />
      </div>
    </div>
  );
}
import React from "react";
import styles from "./styles/MoodTracker.module.css";

const moods = [
  { key: "excited", icon: "🤩", label: "Excited" },
  { key: "happy", icon: "😊", label: "Happy" },
  { key: "neutral", icon: "😐", label: "Neutral" },
  { key: "sad", icon: "😢", label: "Sad" },
  { key: "angry", icon: "😡", label: "Angry" }
];

export default function MoodTracker({ mood, setMood }) {
  return (
    <div className={styles.moodTracker}>
      <h3>Mood Tracker</h3>
      <div className={styles.moodRow}>
        {moods.map(m => (
          <button
            key={m.key}
            className={mood === m.key ? styles.selected : ""}
            onClick={() => setMood(m.key)}
          >
            <span role="img" aria-label={m.label}>
              {m.icon}
            </span>
            <span>{m.label}</span>
          </button>
        ))}
      </div>
    </div>
  );
}
import React, { useState } from "react";
import styles from "./styles/HabitTracker.module.css";

export default function HabitTracker({ habits, setHabits }) {
  const [habitInput, setHabitInput] = useState("");
  function addHabit() {
    if (habitInput.trim() === "") return;
    setHabits([...habits, { name: habitInput, streak: 0, doneToday: false }]);
    setHabitInput("");
  }
  function toggleHabit(idx) {
    setHabits(
      habits.map((h, i) =>
        i === idx
          ? {
              ...h,
              doneToday: !h.doneToday,
              streak: !h.doneToday ? h.streak + 1 : 0
            }
          : h
      )
    );
  }

  return (
    <div className={styles.habitTracker}>
      <h3>Habit Tracker</h3>
      <div className={styles.inputRow}>
        <input
          type="text"
          value={habitInput}
          placeholder="New habit..."
          onChange={e => setHabitInput(e.target.value)}
        />
        <button onClick={addHabit}>Add</button>
      </div>
      <ul>
        {habits.map((h, i) => (
          <li key={i}>
            <input
              type="checkbox"
              checked={h.doneToday}
              onChange={() => toggleHabit(i)}
            />
            {h.name} <span className={styles.streak}>🔥{h.streak}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}
.dashboard {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem;
}
.sections {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}
.overview {
  background: #f9f9f9;
  border-radius: 12px;
  padding: 1rem 2rem;
  margin-bottom: 2rem;
  font-size: 1.1rem;
  display: flex;
  gap: 2rem;
  align-items: center;
}
.timer {
  background: #fff8e1;
  border-radius: 12px;
  padding: 1.5rem;
  text-align: center;
}
.icon {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  width: 50px;
  height: 50px;
  background: #ffeb3b;
  border-radius: 50%;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.durations button {
  margin: 0 4px;
  padding: 0.5em 1em;
  border-radius: 8px;
  border: none;
  background: #ffe082;
  cursor: pointer;
}
.selected {
  background: #ffd54f !important;
  font-weight: bold;
}
.display {
  font-size: 2rem;
  margin: 1rem 0;
  letter-spacing: 2px;
}
.controls button {
  padding: 0.5em 2em;
  background: #ffb300;
  border: none;
  border-radius: 8px;
  color: #fff;
  font-weight: bold;
}
.journal {
  background: #f3f6fc;
  border-radius: 12px;
  padding: 1.5rem;
}
textarea {
  width: 100%;
  min-height: 50px;
  margin-bottom: 1em;
}
.star,
.starActive {
  font-size: 1.5em;
  cursor: pointer;
  color: #bbb;
  margin-right: 0.1em;
}
.starActive {
  color: #ffd700;
}
input[type="text"] {
  margin-left: 0.5em;
  width: 60%;
  padding: 0.25em;
}
ul {
  padding-left: 1em;
}
button {
  margin-top: 0.5em;
}
.moodTracker {
  background: #f0fff0;
  border-radius: 12px;
  padding: 1.5rem;
}
.moodRow {
  display: flex;
  gap: 1em;
  margin-top: 1em;
}
button {
  background: #e0f7fa;
  border: none;
  border-radius: 8px;
  padding: 0.8em 1em;
  font-size: 1.2em;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: background 0.2s;
}
.selected {
  background: #00bcd4 !important;
  color: #fff;
}
.habitTracker {
  background: #fbeffb;
  border-radius: 12px;
  padding: 1.5rem;
}
.inputRow {
  display: flex;
  gap: 0.5em;
  margin-bottom: 1em;
}
input[type="text"] {
  flex: 1;
  padding: 0.4em;
}
.streak {
  margin-left: 0.5em;
  color: #e57373;
  font-weight: bold;
}
ul {
  padding-left: 1em;
}
