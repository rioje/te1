# te1

write song


import React, { useState, useEffect, useRef, useCallback } from 'react';
import * as Tone from 'tone';
import Keyboard from './components/Keyboard';
import DynamicVisualizer from './components/DynamicVisualizer';
import AudioVisualizer from './components/AudioVisualizer';
import
'./App.css';

const TRANSACTIONS = [
    { name: "Hemi", color: "green", note: "C4", soundType: "C Major" },
    { name: "Pop", color: "blue", note: "G4", soundType: "G Major" },
    { name: "ETH", color: "orange", note: "A4", soundType: "A Minor" },
    { name: "New Block", color: "purple", note: "F4", soundType: "F Major" },
];

function App() {
    const [transactions, setTransactions] = useState([]);
    const [soundType, setSoundType] = useState('classical');
    const [mode, setMode] = useState('sequential');
    const analyser = useRef(new Tone.Analyser("fft", 64));

    // 确保 Tone.js 音频上下文启动
    const handleStartAudioContext = async () => {
        await Tone.start();
        console.log("Audio context started");
    };

    const playSound = useCallback((note) => {
        let synth;
        switch (soundType) {
            case 'classical':
                synth = new Tone.Synth().connect(analyser.current);
                break;
            case 'casio':
                synth = new Tone.AMSynth().connect(analyser.current);
                break;
            case 'organ':
                synth = new Tone.FMSynth().connect(analyser.current);
                break;
            default:
                synth = new Tone.Synth().connect(analyser.current);
        }
        synth.triggerAttackRelease(note, "8n");
    }, [soundType]);

    useEffect(() => {
        const interval = setInterval(() => {
            const randomTransaction = TRANSACTIONS[Math.floor(Math.random() * TRANSACTIONS.length)];
            const newTransaction = {
                ...randomTransaction,
                time: new Date().toLocaleTimeString(),
            };
            setTransactions((prev) => [newTransaction, ...prev.slice(0, 9)]);
            playSound(newTransaction.note);
        }, 2000);

        return () => clearInterval(interval);
    }, [mode, playSound]);

    return (
        <div className="App">
            <header className="App-header">
                <div className="transaction-types">
                    {TRANSACTIONS.map((transaction) => (
                        <div key={transaction.name} className="transaction-indicator" style={{ color: transaction.color }}>
                            {transaction.name}
                        </div>
                    ))}
                </div>
                <button onClick={handleStartAudioContext} className="connect-wallet">
                    Start Audio
                </button>
            </header>

            <div className="controls">
                <label>Sound Type: </label>
                <select value={soundType} onChange={(e) => setSoundType(e.target.value)}>
                    <option value="classical">Classical Piano</option>
                    <option value="casio">Casio Piano</option>
                    <option value="organ">Organ</option>
                </select>

                <label>Mode: </label>
                <select value={mode} onChange={(e) => setMode(e.target.value)}>
                    <option value="sequential">Sequential</option>
                    <option value="batch">Batch</option>
                </select>
            </div>

            <div className="visualizers">
                <AudioVisualizer frequencyData={analyser.current} />
                <DynamicVisualizer transactions={transactions} />
            </div>
            
            <main>
                <Keyboard transactions={transactions} soundType={soundType} />
            </main>
        </div>
    );
}


export default App;

