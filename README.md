# Lucky-numbers
Une application générant deux numéros de chance par jour. 
import React, { useState, useEffect } from 'react';
import { Sparkles, Calendar, Settings, X } from 'lucide-react';

const LuckyNumbersApp = () => {
  const [birthDay, setBirthDay] = useState('');
  const [birthMonth, setBirthMonth] = useState('');
  const [birthYear, setBirthYear] = useState('');
  const [numbers, setNumbers] = useState([]);
  const [isGenerating, setIsGenerating] = useState(false);
  const [currentDate, setCurrentDate] = useState('');
  const [hasGenerated, setHasGenerated] = useState(false);
  const [numberOfBalls, setNumberOfBalls] = useState(2);
  const [showSettings, setShowSettings] = useState(false);
  const [maxNumber, setMaxNumber] = useState(90);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    loadSavedData();
  }, []);

  const loadSavedData = async () => {
    try {
      const savedData = await window.storage.get('lucky-numbers-data');
      if (savedData) {
        const data = JSON.parse(savedData.value);
        setBirthDay(data.birthDay);
        setBirthMonth(data.birthMonth);
        setBirthYear(data.birthYear);
        setNumberOfBalls(data.numberOfBalls || 2);
        setMaxNumber(data.maxNumber || 90);
        
        const todayKey = getTodayKey();
        if (data.dateKey === todayKey && data.numbers) {
          setNumbers(data.numbers);
          setCurrentDate(data.currentDate);
          setHasGenerated(true);
        } else if (data.birthDay && data.birthMonth && data.birthYear) {
          generateNumbersForDate(data.birthDay, data.birthMonth, data.birthYear, data.numberOfBalls || 2, data.maxNumber || 90);
        }
      }
    } catch (error) {
      console.log('Aucune donnée sauvegardée trouvée');
    } finally {
      setIsLoading(false);
    }
  };

  const saveData = async (data) => {
    try {
      await window.storage.set('lucky-numbers-data', JSON.stringify(data));
    } catch (error) {
      console.error('Erreur lors de la sauvegarde:', error);
    }
  };

  const getTodayKey = () => {
    const today = new Date();
    return `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
  };

  const isValidDate = (day, month, year) => {
    const d = parseInt(day);
    const m = parseInt(month);
    const y = parseInt(year);
    
    if (m < 1 || m > 12) return false;
    if (y < 1900 || y > new Date().getFullYear()) return false;
    
    const daysInMonth = new Date(y, m, 0).getDate();
    if (d < 1 || d > daysInMonth) return false;
    
    return true;
  };

  const generateDailyNumbers = (dateKey, birthDate, count, max) => {
    const combinedKey = dateKey + birthDate;
    let seed = 0;
    
    for (let i = 0; i < combinedKey.length; i++) {
      seed = ((seed << 5) - seed) + combinedKey.charCodeAt(i);
      seed = seed & seed;
    }
    
    const random = (maximum) => {
      seed = Math.abs((seed * 9301 + 49297) % 233280);
      return Math.floor((seed / 233280) * maximum) + 1;
    };
    
    const generatedNumbers = [];
    let attempts = 0;
    const maxAttempts = max * 10;
    
    while (generatedNumbers.length < count && attempts < maxAttempts) {
      const num = random(max);
      if (!generatedNumbers.includes(num)) {
        generatedNumbers.push(num);
      }
      attempts++;
    }
    
    return generatedNumbers;
  };

  const generateNumbersForDate = (day, month, year, count, max) => {
    const todayKey = getTodayKey();
    const birthDate = `${year}-${String(month).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
    const date = new Date().toLocaleDateString('fr-FR');
    
    setCurrentDate(date);
    setIsGenerating(true);
    
    setTimeout(() => {
      const dailyNumbers = generateDailyNumbers(todayKey, birthDate, count, max);
      setNumbers(dailyNumbers);
      setIsGenerating(false);
      setHasGenerated(true);
      
      saveData({
        birthDay: day,
        birthMonth: month,
        birthYear: year,
        numbers: dailyNumbers,
        currentDate: date,
        dateKey: todayKey,
        numberOfBalls: count,
        maxNumber: max
      });
    }, 1000);
  };

  const handleGenerateNumbers = () => {
    if (!birthDay || !birthMonth || !birthYear) {
      alert('Veuillez entrer votre date de naissance complète');
      return;
    }

    if (!isValidDate(birthDay, birthMonth, birthYear)) {
      alert('Veuillez entrer une date de naissance valide');
      return;
    }

    generateNumbersForDate(birthDay, birthMonth, birthYear, numberOfBalls, maxNumber);
  };

  const handleRegenerateWithSettings = () => {
    if (birthDay && birthMonth && birthYear) {
      generateNumbersForDate(birthDay, birthMonth, birthYear, numberOfBalls, maxNumber);
      setShowSettings(false);
    }
  };

  useEffect(() => {
    if (hasGenerated && birthDay && birthMonth && birthYear) {
      const checkMidnight = () => {
        const todayKey = getTodayKey();
        const storedDateParts = currentDate.split('/');
        const storedDateKey = `${storedDateParts[2]}-${storedDateParts[1].padStart(2, '0')}-${storedDateParts[0].padStart(2, '0')}`;
        
        if (todayKey !== storedDateKey) {
          generateNumbersForDate(birthDay, birthMonth, birthYear, numberOfBalls, maxNumber);
        }
      };
      
      const interval = setInterval(checkMidnight, 60000);
      return () => clearInterval(interval);
    }
  }, [currentDate, hasGenerated, birthDay, birthMonth, birthYear, numberOfBalls, maxNumber]);

  if (isLoading) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-blue-900 via-blue-700 to-blue-500 flex items-center justify-center">
        <div className="text-white text-xl">Chargement...</div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-900 via-blue-700 to-blue-500 flex items-center justify-center p-4">
      <div className="max-w-sm w-full bg-white rounded-3xl shadow-2xl p-6 relative">
        {hasGenerated && (
          <button
            onClick={() => setShowSettings(!showSettings)}
            className="absolute top-4 right-4 p-2 text-blue-600 hover:bg-blue-50 rounded-full transition-colors"
          >
            {showSettings ? <X className="w-6 h-6" /> : <Settings className="w-6 h-6" />}
          </button>
        )}

        <div className="text-center mb-6">
          <div className="flex items-center justify-center mb-2">
            <Sparkles className="w-6 h-6 text-blue-600 mr-2" />
            <h1 className="text-2xl font-bold text-blue-900">Numéros de Chance</h1>
            <Sparkles className="w-6 h-6 text-blue-600 ml-2" />
          </div>
          <p className="text-blue-600 text-sm font-medium">Vos numéros personnalisés du jour</p>
        </div>

        {showSettings && hasGenerated && (
          <div className="mb-4 bg-blue-50 border-2 border-blue-200 rounded-xl p-3 space-y-3">
            <h3 className="font-bold text-blue-900 mb-2 text-sm">Paramètres</h3>
            
            <div>
              <label className="block text-gray-700 font-medium mb-2">
                Two sure
              </label>
              <input
                type="number"
                min="2"
                max="10"
                value={numberOfBalls}
                onChange={(e) => setNumberOfBalls(Math.min(10, Math.max(2, parseInt(e.target.value) || 2)))}
                className="w-full px-4 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-center"
              />
            </div>

            <div>
              <label className="block text-gray-700 font-medium mb-2">
                Numéro maximum (1-90)
              </label>
              <input
                type="number"
                min="1"
                max="90"
                value={maxNumber}
                onChange={(e) => setMaxNumber(Math.min(90, Math.max(1, parseInt(e.target.value) || 90)))}
                className="w-full px-4 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-center"
              />
            </div>

            <button
              onClick={handleRegenerateWithSettings}
              className="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition-colors"
            >
              Appliquer les paramètres
            </button>
          </div>
        )}

        {!hasGenerated ? (
          <div className="space-y-4">
            <div className="bg-blue-50 border border-blue-200 rounded-lg p-3 mb-4">
              <div className="flex items-center justify-center mb-1">
                <Calendar className="w-4 h-4 text-blue-700 mr-2" />
                <p className="text-blue-800 font-semibold text-sm">Entrez votre date de naissance</p>
              </div>
              <p className="text-blue-600 text-xs text-center">
                Pour générer vos numéros de chance personnalisés
              </p>
            </div>

            <div className="space-y-3">
              <div>
                <label className="block text-gray-700 font-medium mb-1 text-sm">Jour</label>
                <input
                  type="number"
                  min="1"
                  max="31"
                  placeholder="JJ"
                  value={birthDay}
                  onChange={(e) => setBirthDay(e.target.value)}
                  className="w-full px-3 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-base text-center"
                />
              </div>

              <div>
                <label className="block text-gray-700 font-medium mb-1 text-sm">Mois</label>
                <input
                  type="number"
                  min="1"
                  max="12"
                  placeholder="MM"
                  value={birthMonth}
                  onChange={(e) => setBirthMonth(e.target.value)}
                  className="w-full px-3 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-base text-center"
                />
              </div>

              <div>
                <label className="block text-gray-700 font-medium mb-1 text-sm">Année</label>
                <input
                  type="number"
                  min="1900"
                  max={new Date().getFullYear()}
                  placeholder="AAAA"
                  value={birthYear}
                  onChange={(e) => setBirthYear(e.target.value)}
                  className="w-full px-3 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-base text-center"
                />
              </div>
            </div>

            <div className="space-y-3 pt-3 border-t border-blue-200">
              <div>
                <label className="block text-gray-700 font-medium mb-2">
                  Two sure
                </label>
                <input
                  type="number"
                  min="2"
                  max="10"
                  value={numberOfBalls}
                  onChange={(e) => setNumberOfBalls(Math.min(10, Math.max(2, parseInt(e.target.value) || 2)))}
                  className="w-full px-4 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-center"
                />
              </div>

              <div>
                <label className="block text-gray-700 font-medium mb-2">
                  Numéro maximum (1-90)
                </label>
                <input
                  type="number"
                  min="1"
                  max="90"
                  value={maxNumber}
                  onChange={(e) => setMaxNumber(Math.min(90, Math.max(1, parseInt(e.target.value) || 90)))}
                  className="w-full px-4 py-2 border-2 border-blue-200 rounded-lg focus:border-blue-500 focus:outline-none text-center"
                />
              </div>
            </div>

            <button
              onClick={handleGenerateNumbers}
              disabled={isGenerating}
              className="w-full bg-gradient-to-r from-blue-600 to-blue-700 hover:from-blue-700 hover:to-blue-800 text-white font-bold py-3 px-4 rounded-xl shadow-lg transform transition-all duration-200 hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed text-sm"
            >
              {isGenerating ? 'Génération...' : 'Générer mes numéros de chance'}
            </button>
          </div>
        ) : (
          <div>
            <div className="text-center mb-4">
              <p className="text-gray-600 text-xs mb-1">Date de naissance</p>
              <p className="text-blue-700 font-bold text-sm">{birthDay}/{birthMonth}/{birthYear}</p>
              <p className="text-gray-600 text-xs mt-2 mb-1">Numéros du</p>
              <p className="text-blue-700 font-bold text-base">{currentDate}</p>
            </div>

            <div className={`grid ${numberOfBalls <= 3 ? 'grid-cols-2' : numberOfBalls <= 6 ? 'grid-cols-3' : 'grid-cols-4'} gap-3 mb-6 justify-items-center`}>
              {numbers.map((num, index) => (
                <div
                  key={index}
                  className={`relative ${numberOfBalls <= 3 ? 'w-24 h-24' : numberOfBalls <= 6 ? 'w-20 h-20' : 'w-16 h-16'} rounded-full bg-gradient-to-br from-blue-400 to-blue-600 shadow-xl flex items-center justify-center transform transition-all duration-500 ${
                    isGenerating ? 'scale-0 rotate-180' : 'scale-100 rotate-0'
                  }`}
                  style={{ transitionDelay: `${index * 100}ms` }}
                >
                  <div className={`absolute ${numberOfBalls <= 3 ? 'top-2 left-4 w-6 h-6' : numberOfBalls <= 6 ? 'top-2 left-3 w-5 h-5' : 'top-1 left-2 w-4 h-4'} bg-white opacity-30 rounded-full blur-sm`}></div>
                  <span className={`${numberOfBalls <= 3 ? 'text-3xl' : numberOfBalls <= 6 ? 'text-2xl' : 'text-xl'} font-bold text-white z-10`}>{num}</span>
                  <div className="absolute inset-0 rounded-full shadow-inner"></div>
                </div>
              ))}
            </div>

            <div className="bg-blue-50 border border-blue-200 rounded-lg p-3 text-center mb-3">
              <p className="text-blue-800 text-xs">
                Ces numéros sont basés sur votre date de naissance et la date du jour. Ils changeront automatiquement à minuit. 🌙
              </p>
            </div>

            <button
              onClick={() => {
                setHasGenerated(false);
                setNumbers([]);
                setShowSettings(false);
              }}
              className="w-full bg-gray-200 hover:bg-gray-300 text-gray-700 font-medium py-2 px-4 rounded-lg transition-colors duration-200 text-sm"
            >
              Changer la date de naissance
            </button>

            <div className="mt-3 text-center">
              <p className="text-blue-700 font-medium text-xs">
                🍀 Bonne chance avec vos numéros ! 🍀
              </p>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

export default LuckyNumbersApp;
