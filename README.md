# exams
import React, { useState, useEffect } from 'react';
import { 
  Folder, FolderPlus, FileText, Plus, Trash2, Edit3, ChevronRight, ChevronDown, 
  Play, BookOpen, CheckCircle, XCircle, ArrowLeft, Download, Upload, Video, 
  Sparkles, Clock, Award, Globe, Layers, AlertCircle, Save, Check, RefreshCw, Printer, FileDown
} from 'lucide-react';

// --- INITIAL MOCK DATA ---
const initialFolders = [
  {
    id: 'c1',
    name: '1º Bachillerato - Biología y Geología',
    type: 'course',
    children: [
      {
        id: 't1',
        name: '1er Trimestre',
        type: 'trimester',
        children: [
          {
            id: 'm1',
            name: 'Tema 1: La Célula, Estructura y Función',
            type: 'topic',
            exams: [
              {
                id: 'ex1',
                title: 'Evaluación Diagnóstica: Citología y Organelos',
                description: 'Examen de prueba sobre la ultraestructura celular, transporte a través de membrana y ciclo celular.',
                createdAt: '2026-10-24',
                questions: [
                  {
                    id: 1,
                    type: 'mc',
                    question: '¿Cuál es la función metabólica primordial de las mitocondrias en células eucariotas?',
                    options: [
                      'Síntesis y ensamblaje de ribosomas',
                      'Generación de ATP a través del proceso de respiración celular',
                      'Digestión enzimática de polisacáridos complejos',
                      'Almacenamiento de material genético cromosómico'
                    ],
                    correctAnswer: 1,
                    explanation: 'Las mitocondrias son los orgánulos celulares encargados de la fosforilación oxidativa y síntesis de ATP a partir de nutrientes.'
                  },
                  {
                    id: 2,
                    type: 'tf',
                    question: 'Las células procariotas carecen de envoltura nuclear y organelos delimitados por membranas.',
                    correctAnswer: true,
                    explanation: 'Verdadero. El material genético procariota flota en el citoplasma dentro de la región nucleoide sin membrana nuclear.'
                  },
                  {
                    id: 3,
                    type: 'short',
                    question: '¿Qué organelo contiene hidrolasas ácidas para el reciclaje y degradación intracelular?',
                    correctAnswer: 'Lisosoma',
                    explanation: 'Los lisosomas actúan como el sistema digestivo de la célula degradando materiales sobrantes o nocivos.'
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
];

export default function App() {
  // --- STATE MANAGEMENT ---
  const [folders, setFolders] = useState(() => {
    const saved = localStorage.getItem('deep_rose_white_exam_folders');
    return saved ? JSON.parse(saved) : initialFolders;
  });

  const [selectedTopic, setSelectedTopic] = useState(null);
  const [selectedExam, setSelectedExam] = useState(null);
  const [activeTab, setActiveTab] = useState('library'); // 'library', 'generator', 'practice', 'deployment'
  const [expandedFolders, setExpandedFolders] = useState({ c1: true, t1: true });

  // Generator State
  const [inputSourceType, setInputSourceType] = useState('text'); // 'text', 'file', 'video'
  const [inputText, setInputText] = useState('');
  const [videoUrl, setVideoUrl] = useState('');
  const [numQuestions, setNumQuestions] = useState(5);
  const [difficulty, setDifficulty] = useState('media');
  const [questionTypes, setQuestionTypes] = useState({ mc: true, tf: true, short: true });
  const [examTitle, setExamTitle] = useState('');
  const [selectedTargetTopicId, setSelectedTargetTopicId] = useState('');
  const [isGenerating, setIsGenerating] = useState(false);

  // Manual Editor State
  const [editingExam, setEditingExam] = useState(null);

  // Practice Mode State
  const [userAnswers, setUserAnswers] = useState({});
  const [submitted, setSubmitted] = useState(false);
  const [timer, setTimer] = useState(0);
  const [isTimerRunning, setIsTimerRunning] = useState(false);

  // Modal State for Folders
  const [modalOpen, setModalOpen] = useState(false);
  const [modalType, setModalType] = useState('course'); // 'course', 'trimester', 'topic'
  const [modalParentId, setModalParentId] = useState(null);
  const [newItemName, setNewItemName] = useState('');

  // Persist State
  useEffect(() => {
    localStorage.setItem('deep_rose_white_exam_folders', JSON.stringify(folders));
  }, [folders]);

  // Timer logic
  useEffect(() => {
    let interval;
    if (isTimerRunning) {
      interval = setInterval(() => {
        setTimer((prev) => prev + 1);
      }, 1000);
    } else {
      clearInterval(interval);
    }
    return () => clearInterval(interval);
  }, [isTimerRunning]);

  // Toggle Folder
  const toggleFolder = (id) => {
    setExpandedFolders(prev => ({ ...prev, [id]: !prev[id] }));
  };

  // Add Item to Hierarchy
  const handleAddItem = () => {
    if (!newItemName.trim()) return;

    const newId = 'id_' + Date.now();
    const updated = JSON.parse(JSON.stringify(folders));

    if (modalType === 'course') {
      updated.push({
        id: newId,
        name: newItemName,
        type: 'course',
        children: []
      });
    } else if (modalType === 'trimester') {
      const course = updated.find(c => c.id === modalParentId);
      if (course) {
        course.children = course.children || [];
        course.children.push({
          id: newId,
          name: newItemName,
          type: 'trimester',
          children: []
        });
      }
    } else if (modalType === 'topic') {
      for (const c of updated) {
        const trimester = (c.children || []).find(t => t.id === modalParentId);
        if (trimester) {
          trimester.children = trimester.children || [];
          trimester.children.push({
            id: newId,
            name: newItemName,
            type: 'topic',
            exams: []
          });
          break;
        }
      }
    }

    setFolders(updated);
    setNewItemName('');
    setModalOpen(false);
  };

  // Delete Item
  const handleDeleteItem = (id, type) => {
    if (!window.confirm('¿Seguro que deseas eliminar este elemento y todo su contenido?')) return;

    let updated = JSON.parse(JSON.stringify(folders));

    if (type === 'course') {
      updated = updated.filter(c => c.id !== id);
    } else if (type === 'trimester') {
      updated.forEach(c => {
        c.children = (c.children || []).filter(t => t.id !== id);
      });
    } else if (type === 'topic') {
      updated.forEach(c => {
        (c.children || []).forEach(t => {
          t.children = (t.children || []).filter(m => m.id !== id);
        });
      });
    }

    setFolders(updated);
    if (selectedTopic?.id === id) setSelectedTopic(null);
  };

  // Handle File Reading
  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (event) => {
      setInputText(event.target.result);
    };
    reader.readAsText(file);
  };

  // Automatic Exam Generator Logic
  const handleGenerateExam = () => {
    if (!inputText.trim() && !videoUrl.trim()) {
      alert('Por favor, introduce apuntes en texto, adjunta un archivo o un vídeo.');
      return;
    }

    if (!examTitle.trim()) {
      alert('Por favor, asigna un título al examen antes de continuar.');
      return;
    }

    setIsGenerating(true);

    setTimeout(() => {
      const source = inputText || `Análisis de transcripción en vídeo: ${videoUrl}`;
      const sentences = source.split(/(?<=[.?!])\s+/).filter(s => s.trim().length > 12);
      
      const newQuestions = [];
      const types = [];
      if (questionTypes.mc) types.push('mc');
      if (questionTypes.tf) types.push('tf');
      if (questionTypes.short) types.push('short');
      if (types.length === 0) types.push('mc');

      for (let i = 0; i < numQuestions; i++) {
        const qType = types[i % types.length];
        const sentence = sentences[i % sentences.length] || `Concepto clave de estudio #${i + 1}`;

        if (qType === 'mc') {
          newQuestions.push({
            id: i + 1,
            type: 'mc',
            question: `Sobre la materia estudiada: ¿Cuál es el planteamiento exacto relativo a "${sentence.substring(0, 45)}..."?`,
            options: [
              sentence.substring(0, 65) + '.',
              'Constituye una propiedad secundaria sin validez experimental.',
              'Corresponde a una hipótesis obsoleta rechazada en los textos de estudio.',
              'Se verifica de manera aislada en condiciones especiales.'
            ],
            correctAnswer: 0,
            explanation: `Extraído del pasaje de referencia: "${sentence}"`
          });
        } else if (qType === 'tf') {
          const isTrue = i % 2 === 0;
          newQuestions.push({
            id: i + 1,
            type: 'tf',
            question: isTrue 
              ? `¿Es VERDADERO el enunciado: "${sentence}"?` 
              : `¿Es VERDADERO que el texto niega categóricamente que: "${sentence.substring(0, 40)}..."?`,
            correctAnswer: isTrue,
            explanation: `Basado estrictamente en las notas analizadas.`
          });
        } else {
          newQuestions.push({
            id: i + 1,
            type: 'short',
            question: `Explica en tus propias palabras el postulado: "${sentence.substring(0, 40)}..."`,
            correctAnswer: sentence.substring(0, 30),
            explanation: `La respuesta debe argumentar sobre los pilares del texto adjunto.`
          });
        }
      }

      const generatedExam = {
        id: 'ex_' + Date.now(),
        title: examTitle,
        description: `Examen autogenerado. Dificultad: ${difficulty.toUpperCase()}.`,
        createdAt: new Date().toISOString().split('T')[0],
        questions: newQuestions
      };

      setEditingExam(generatedExam);
      setIsGenerating(false);
    }, 1000);
  };

  // Save Exam to Selected Topic
  const handleSaveExam = () => {
    if (!editingExam) return;

    if (selectedTargetTopicId) {
      const updated = JSON.parse(JSON.stringify(folders));
      let added = false;
      for (const c of updated) {
        for (const t of c.children || []) {
          for (const m of t.children || []) {
            if (m.id === selectedTargetTopicId) {
              m.exams = m.exams || [];
              m.exams.push(editingExam);
              added = true;
              setSelectedTopic(m);
              break;
            }
          }
        }
      }
      if (added) setFolders(updated);
    }

    setSelectedExam(editingExam);
    setEditingExam(null);
    setUserAnswers({});
    setSubmitted(false);
    setTimer(0);
    setIsTimerRunning(true);
    setActiveTab('practice');
  };

  // Launch Exam Practice
  const handleStartExam = (exam) => {
    setSelectedExam(exam);
    setUserAnswers({});
    setSubmitted(false);
    setTimer(0);
    setIsTimerRunning(true);
    setActiveTab('practice');
  };

  // Compute Grade Score
  const calculateScore = () => {
    if (!selectedExam) return { total: 0, correct: 0, percentage: 0, mark: 0 };
    let correct = 0;
    selectedExam.questions.forEach((q) => {
      const userAns = userAnswers[q.id];
      if (q.type === 'mc' && userAns === q.correctAnswer) correct++;
      if (q.type === 'tf' && userAns === q.correctAnswer) correct++;
      if (q.type === 'short' && userAns?.trim().toLowerCase() === q.correctAnswer.toLowerCase()) correct++;
    });
    const percentage = Math.round((correct / selectedExam.questions.length) * 100);
    const mark = ((correct / selectedExam.questions.length) * 10).toFixed(1);
    return { total: selectedExam.questions.length, correct, percentage, mark };
  };

  // Time formatter
  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  // Export JSON Backup
  const handleExportJSON = () => {
    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(folders, null, 2));
    const downloadAnchor = document.createElement('a');
    downloadAnchor.setAttribute("href", dataStr);
    downloadAnchor.setAttribute("download", `examenes_rosa_frances_oscuro.json`);
    document.body.appendChild(downloadAnchor);
    downloadAnchor.click();
    downloadAnchor.remove();
  };

  // Import JSON Backup
  const handleImportJSON = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (event) => {
      try {
        const importedData = JSON.parse(event.target.result);
        if (Array.isArray(importedData)) {
          setFolders(importedData);
          alert('¡Cursos, temas y exámenes importados correctamente!');
        } else {
          alert('Formato de archivo JSON no compatible.');
        }
      } catch (err) {
        alert('Error al leer la copia de seguridad.');
      }
    };
    reader.readAsText(file);
  };

  // Helper: List all topics
  const getAllTopics = () => {
    const list = [];
    folders.forEach(c => {
      (c.children || []).forEach(t => {
        (t.children || []).forEach(m => {
          list.push({ id: m.id, name: `${c.name} ➔ ${t.name} ➔ ${m.name}` });
        });
      });
    });
    return list;
  };

  return (
    <div className="min-h-screen bg-[#ffffff] text-[#1a0910] font-serif flex flex-col selection:bg-[#ffccd5] selection:text-[#590d22]">
      <style>{`
        body, button, input, textarea, select {
          font-family: 'Georgia', 'Times New Roman', serif;
        }
        @media print {
          .no-print { display: none !important; }
          .print-only { display: block !important; }
          body { background: white !important; color: black !important; }
          .printable-card { border: none !important; shadow: none !important; padding: 0 !important; }
        }
      `}</style>

      {/* HEADER */}
      <header className="bg-gradient-to-r from-[#590d22] via-[#800f2f] to-[#a4133c] text-white shadow-md border-b border-[#ffb3c6] no-print">
        <div className="max-w-7xl mx-auto px-4 py-3.5 flex flex-wrap justify-between items-center gap-4">
          <div className="flex items-center space-x-3">
            <div className="bg-white/10 p-2 rounded-2xl border border-white/20 backdrop-blur-sm shadow-inner">
              <BookOpen className="w-6 h-6 text-[#ffccd5]" />
            </div>
            <div>
              <h1 className="text-2xl font-bold tracking
