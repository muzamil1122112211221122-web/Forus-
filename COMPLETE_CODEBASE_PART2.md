# Forus Heavy API - Complete Codebase (Part 2)

## CLIENT PAGES

### client/src/pages/landing.tsx
```typescript
import { useLocation } from "wouter";
import { Button } from "@/components/ui/button";
import { Hammer } from "lucide-react";

export default function Landing() {
  const [, setLocation] = useLocation();

  const handleStart = () => {
    setLocation("/chat");
  };

  const handleSkip = () => {
    setLocation("/chat");
  };

  return (
    <div className="font-sans bg-white text-black min-h-screen flex flex-col overflow-hidden">
      {/* Grid Background */}
      <div className="fixed inset-0 grid-background"></div>
      <div className="fixed inset-0 grid-overlay"></div>
      
      {/* Skip Button */}
      <div className="absolute top-4 right-4 z-20 sm:top-6 sm:right-6">
        <button 
          onClick={handleSkip}
          className="text-gray-500 hover:text-black text-sm sm:text-base transition-colors duration-200 px-3 py-2"
        >
          Skip
        </button>
      </div>

      {/* Main Content Container */}
      <div className="relative z-10 flex-1 flex flex-col items-center justify-center px-4 sm:px-6 lg:px-8">
        
        {/* Hammer Icon and Title */}
        <div className="text-center mb-6 sm:mb-8">
          <div className="mb-4">
            <Hammer className="text-black animate-bounce mx-auto" style={{width: '58px', height: '58px'}} />
          </div>
          <h1 className="title-responsive lg:text-5xl xl:text-6xl font-bold text-black mb-3 sm:mb-4">
            Forus Heavy API
          </h1>
          <p className="tagline-responsive sm:text-lg lg:text-xl text-gray-600 max-w-2xl mx-auto leading-relaxed">
            Forus core, world's first dual api integrated ai
          </p>
        </div>

        {/* Start Button */}
        <div className="mb-8 sm:mb-12">
          <Button 
            onClick={handleStart}
            className="bg-gray-100 border border-gray-300 text-black px-8 sm:px-12 py-3 sm:py-4 rounded-full text-base sm:text-lg font-medium hover:bg-gray-200 hover:border-gray-400 transition-all duration-300 hover:-translate-y-0.5"
          >
            Start
          </Button>
        </div>

      </div>

      {/* Footer */}
      <footer className="relative z-10 pb-6 sm:pb-8 px-4 sm:px-6 space-y-4">
        
        {/* Terms and Privacy */}
        <div className="text-center">
          <p className="text-xs sm:text-sm text-gray-500 leading-relaxed">
            By continuing you agree to the{" "}
            <a href="#" className="text-gray-600 hover:text-black underline transition-colors">
              Terms of Service
            </a>{" "}
            and{" "}
            <a href="#" className="text-gray-600 hover:text-black underline transition-colors">
              Privacy Policy
            </a>
          </p>
        </div>

        {/* Attribution */}
        <div className="text-right">
          <p className="text-sm text-gray-500 italic">Made by Muzamil</p>
        </div>

      </footer>
    </div>
  );
}
```

### client/src/pages/main-chat.tsx
```typescript
import { useState, useRef, useEffect, useCallback } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card } from "@/components/ui/card";
import { ScrollArea } from "@/components/ui/scroll-area";
import { useWebSocket } from "@/hooks/use-websocket";
import { useSpeech } from "@/hooks/use-speech";
import { Send, Mic, MicOff, Image, FileText, Volume2, VolumeX, Settings, Hammer, ThumbsUp, ThumbsDown, Copy, X, Upload } from "lucide-react";
import { ThemeToggle } from "@/components/theme-toggle";

interface Message {
  id: string;
  content: string;
  isUser: boolean;
  timestamp: Date;
  type?: 'text' | 'image' | 'file';
}

export default function MainChat() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState("");
  const [isTyping, setIsTyping] = useState(false);
  const [isSpeaking, setIsSpeaking] = useState(false);
  const [hasWelcomeMessage, setHasWelcomeMessage] = useState(false);
  const [showSettings, setShowSettings] = useState(false);
  const [selectedFiles, setSelectedFiles] = useState<File[]>([]);
  const [showFilePreview, setShowFilePreview] = useState(false);
  const [isDualMode, setIsDualMode] = useState(false);
  const [currentResponse, setCurrentResponse] = useState<string>("");
  const [likedMessages, setLikedMessages] = useState<Set<string>>(new Set());
  const [dislikedMessages, setDislikedMessages] = useState<Set<string>>(new Set());
  const [copiedMessage, setCopiedMessage] = useState<string | null>(null);
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const { 
    isRecording, 
    startRecording, 
    stopRecording, 
    speak, 
    stopSpeaking,
    isSupported: speechSupported 
  } = useSpeech({
    onTranscript: (transcript) => {
      setInput(transcript);
    },
    onSpeakStart: () => setIsSpeaking(true),
    onSpeakEnd: () => setIsSpeaking(false)
  });

  const handleWebSocketMessage = useCallback((data: any) => {
    if (data.type === 'message') {
      // Check if this is a welcome message and we already have one
      const isWelcomeMessage = data.content?.includes('Welcome to Forus Heavy API');
      if (isWelcomeMessage && hasWelcomeMessage) {
        return; // Skip duplicate welcome messages
      }
      
      setMessages(prev => [...prev, {
        id: `${Date.now()}-${Math.random()}`,
        content: data.content || '',
        isUser: false,
        timestamp: new Date(),
        type: data.messageType || 'text'
      }]);
      
      if (isWelcomeMessage) {
        setHasWelcomeMessage(true);
      }
      
      setIsTyping(false);
      
      // Auto-speak AI responses (but not welcome messages)
      if (data.content && !isSpeaking && speak && !isWelcomeMessage) {
        speak(data.content);
      }
    } else if (data.type === 'typing') {
      setIsTyping(data.isTyping || false);
    }
  }, [isSpeaking, speak, hasWelcomeMessage]);

  const { sendMessage, isConnected } = useWebSocket({
    onMessage: handleWebSocketMessage
  });

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const handleSendMessage = (content: string, type: 'text' | 'image' | 'file' = 'text') => {
    if (!content.trim() && type === 'text') return;

    // Add dual integration prefix if dual mode is active
    let finalContent = content;
    if (isDualMode && type === 'text') {
      finalContent = `[DUAL_INTEGRATION] ${content}`;
    }

    const userMessage: Message = {
      id: `${Date.now()}-${Math.random()}`,
      content,
      isUser: true,
      timestamp: new Date(),
      type
    };

    setMessages(prev => [...prev, userMessage]);
    setInput("");
    setIsTyping(true);

    sendMessage({
      content: finalContent,
      type,
      timestamp: new Date().toISOString()
    });
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    handleSendMessage(input);
  };

  const handleFileUpload = (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = Array.from(e.target.files || []);
    if (files.length > 0) {
      setSelectedFiles(files);
      setShowFilePreview(true);
    }
  };

  const uploadSelectedFiles = () => {
    selectedFiles.forEach((file) => {
      const reader = new FileReader();
      reader.onload = (event) => {
        const content = event.target?.result as string;
        handleSendMessage(content, file.type.startsWith('image/') ? 'image' : 'file');
      };
      reader.readAsDataURL(file);
    });
    setSelectedFiles([]);
    setShowFilePreview(false);
  };

  const removeFile = (index: number) => {
    setSelectedFiles(files => files.filter((_, i) => i !== index));
  };

  const handleDualIntegration = () => {
    setIsDualMode(!isDualMode);
  };

  const handleLikeResponse = (messageId: string) => {
    console.log('Liked message:', messageId);
    setLikedMessages(prev => {
      const newSet = new Set(prev);
      if (newSet.has(messageId)) {
        newSet.delete(messageId);
      } else {
        newSet.add(messageId);
        // Remove from disliked if it was disliked
        setDislikedMessages(prevDisliked => {
          const newDislikedSet = new Set(prevDisliked);
          newDislikedSet.delete(messageId);
          return newDislikedSet;
        });
      }
      return newSet;
    });
  };

  const handleDislikeResponse = (messageId: string) => {
    console.log('Disliked message:', messageId);
    setDislikedMessages(prev => {
      const newSet = new Set(prev);
      if (newSet.has(messageId)) {
        newSet.delete(messageId);
      } else {
        newSet.add(messageId);
        // Remove from liked if it was liked
        setLikedMessages(prevLiked => {
          const newLikedSet = new Set(prevLiked);
          newLikedSet.delete(messageId);
          return newLikedSet;
        });
      }
      return newSet;
    });
  };

  const handleCopyResponse = (content: string) => {
    navigator.clipboard.writeText(content);
    setCopiedMessage(content);
    // Clear the copied state after 2 seconds
    setTimeout(() => setCopiedMessage(null), 2000);
  };

  const handleStopResponse = () => {
    setIsTyping(false);
    setCurrentResponse("");
  };

  const toggleRecording = () => {
    if (isRecording) {
      stopRecording();
    } else {
      startRecording();
    }
  };

  const toggleSpeaking = () => {
    if (isSpeaking) {
      stopSpeaking();
    } else {
      const lastAIMessage = messages.filter(m => !m.isUser).pop();
      if (lastAIMessage) {
        speak(lastAIMessage.content);
      }
    }
  };

  return (
    <div className="h-screen bg-white dark:bg-gray-900 text-black dark:text-white flex flex-col">
      {/* Header */}
      <header className="flex items-center justify-between p-4 border-b border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-900">
        <div className="flex items-center space-x-4">
          <div className="w-8 h-8 bg-transparent border-2 border-gray-300 dark:border-gray-600 rounded-xl flex items-center justify-center font-black text-lg">
            <span className="font-black text-xl text-black dark:text-white">ƒ</span>
          </div>
          <h1 className="text-xl font-semibold text-black dark:text-white">Forus Heavy API</h1>
        </div>
        <div className="flex items-center space-x-2">
          <Button variant="outline" size="sm" onClick={() => setInput("Ask me anything...")}>
            Ask
          </Button>

          <ThemeToggle />
          <Button variant="ghost" size="sm" onClick={() => setShowSettings(!showSettings)}>
            <Settings className="w-4 h-4" />
          </Button>
        </div>
      </header>

      {/* Chat Messages Area */}
      <div className="flex-1 flex flex-col items-center justify-center p-4 bg-white dark:bg-gray-900">
        {messages.filter(m => m.isUser).length === 0 ? (
          <div className="text-center max-w-md">
            <div className="w-16 h-16 bg-gray-100 dark:bg-gray-800 rounded-2xl flex items-center justify-center mx-auto mb-6">
              <div className="w-10 h-10 bg-transparent border-2 border-gray-300 dark:border-gray-600 rounded-xl flex items-center justify-center font-black text-xl">
                <span className="font-black text-2xl text-black dark:text-white">ƒ</span>
              </div>
            </div>
            <h2 className="text-2xl font-semibold text-black dark:text-white mb-2">
              Welcome to Forus Heavy API
            </h2>
            <p className="text-gray-600 dark:text-gray-400 mb-8">
              Advanced dual API integrated AI assistant
            </p>
            {/* Show AI welcome messages if any */}
            {messages.length > 0 && (
              <div className="mt-8 max-w-lg">
                {messages.filter(m => !m.isUser).map((message) => (
                  <div
                    key={message.id}
                    className="bg-gray-100 dark:bg-gray-800 text-black dark:text-white border border-gray-300 dark:border-gray-600 p-4 rounded-xl mb-4"
                  >
                    <p className="whitespace-pre-wrap">{message.content}</p>
                    <span className="text-xs opacity-75 mt-2 block">
                      {message.timestamp.toLocaleTimeString()}
                    </span>
                  </div>
                ))}
              </div>
            )}
          </div>
        ) : (
          <ScrollArea className="flex-1 w-full max-w-4xl">
            <div className="space-y-4 p-4">
              {messages.map((message) => (
                <div
                  key={message.id}
                  className={`flex ${message.isUser ? 'justify-end' : 'justify-start'}`}
                >
                  <div
                    className={`max-w-[80%] p-4 rounded-xl border ${
                      message.isUser
                        ? 'bg-blue-600 text-white border-blue-700'
                        : 'bg-gray-100 dark:bg-gray-800 text-black dark:text-white border-gray-300 dark:border-gray-600'
                    }`}
                  >
                    {message.type === 'image' ? (
                      <div>
                        <img src={message.content} alt="Uploaded" className="max-w-full rounded-xl mb-2" />
                        <p>Image uploaded</p>
                      </div>
                    ) : (
                      <p className="whitespace-pre-wrap">{message.content}</p>
                    )}
                    <div className="flex items-center justify-between mt-2">
                      <span className="text-xs opacity-75">
                        {message.timestamp.toLocaleTimeString()}
                      </span>
                      {!message.isUser && (
                        <div className="flex items-center space-x-1">
                          <Button
                            variant="ghost"
                            size="sm"
                            onClick={() => handleLikeResponse(message.id)}
                            className={`h-7 w-7 p-0 hover:bg-transparent transition-colors ${
                              likedMessages.has(message.id) ? 'text-green-500' : 'text-gray-500 hover:text-green-500'
                            }`}
                          >
                            <ThumbsUp className="w-3 h-3" />
                          </Button>
                          <Button
                            variant="ghost"
                            size="sm"
                            onClick={() => handleDislikeResponse(message.id)}
                            className={`h-7 w-7 p-0 hover:bg-transparent transition-colors ${
                              dislikedMessages.has(message.id) ? 'text-red-500' : 'text-gray-500 hover:text-red-500'
                            }`}
                          >
                            <ThumbsDown className="w-3 h-3" />
                          </Button>
                          <Button
                            variant="ghost"
                            size="sm"
                            onClick={() => handleCopyResponse(message.content)}
                            className={`h-7 w-7 p-0 hover:bg-transparent transition-colors ${
                              copiedMessage === message.content ? 'text-blue-500' : 'text-gray-500 hover:text-blue-500'
                            }`}
                          >
                            <Copy className="w-3 h-3" />
                          </Button>
                          {isTyping && (
                            <Button
                              variant="ghost"
                              size="sm"
                              onClick={handleStopResponse}
                              className="h-7 w-7 p-0 hover:bg-orange-100 dark:hover:bg-orange-900"
                            >
                              <X className="w-3 h-3" />
                            </Button>
                          )}
                        </div>
                      )}
                    </div>
                  </div>
                </div>
              ))}
              {isTyping && (
                <div className="flex justify-start">
                  <div className="bg-gray-100 dark:bg-gray-800 border border-gray-300 dark:border-gray-600 p-4 rounded-xl max-w-[80%]">
                    <div className="flex space-x-1 items-center justify-center">
                      <div className="w-3 h-3 bg-gray-500 dark:bg-gray-400 rounded-full typing-dot-1"></div>
                      <div className="w-3 h-3 bg-gray-500 dark:bg-gray-400 rounded-full typing-dot-2"></div>
                      <div className="w-3 h-3 bg-gray-500 dark:bg-gray-400 rounded-full typing-dot-3"></div>
                      <Button
                        variant="ghost"
                        size="sm"
                        onClick={handleStopResponse}
                        className="h-6 w-6 p-0 ml-2 hover:bg-orange-100 dark:hover:bg-orange-900"
                      >
                        <X className="w-3 h-3" />
                      </Button>
                    </div>
                  </div>
                </div>
              )}
              <div ref={messagesEndRef} />
            </div>
          </ScrollArea>
        )}
      </div>

      {/* Bottom Input Bar */}
      <div className="p-4 border-t border-gray-200 dark:border-gray-700 bg-white dark:bg-gray-900">
        <div className="max-w-4xl mx-auto">
          {/* Function Bar - Always Visible */}
          <div className="flex justify-center items-center space-x-2 mb-3 p-3 bg-gray-50 dark:bg-gray-800 border border-gray-200 dark:border-gray-600 rounded-xl overflow-x-auto function-bar-mobile">
            <input
              type="file"
              ref={fileInputRef}
              onChange={handleFileUpload}
              className="hidden"
              accept="image/*,.pdf,.doc,.docx,.txt"
              multiple
            />
            
            <button
              type="button"
              onClick={() => fileInputRef.current?.click()}
              className="flex items-center space-x-1 px-2 py-2 rounded-lg border border-gray-300 dark:border-gray-500 bg-white dark:bg-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600 transition-colors min-w-fit"
              title="Upload Image or Document"
            >
              <Upload className="w-5 h-5 function-bar-icon" />
              <span className="text-xs font-medium text-gray-700 dark:text-gray-200">Upload</span>
            </button>

            <button
              type="button"
              onClick={toggleRecording}
              className={`flex items-center space-x-1 px-2 py-2 rounded-lg border transition-colors min-w-fit ${
                isRecording 
                  ? 'border-red-500 bg-red-50 dark:bg-red-900/20' 
                  : 'border-gray-300 dark:border-gray-500 bg-white dark:bg-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600'
              }`}
              title={isRecording ? "Stop Recording" : "Start Voice Input"}
            >
              {isRecording ? <MicOff className="w-5 h-5 text-red-500 function-bar-icon" /> : <Mic className="w-5 h-5 function-bar-icon" />}
              <span className={`text-xs font-medium ${isRecording ? 'text-red-600 dark:text-red-400' : 'text-gray-700 dark:text-gray-200'}`}>
                {isRecording ? 'Stop' : 'Voice'}
              </span>
            </button>

            <button
              type="button"
              onClick={toggleSpeaking}
              className={`flex items-center space-x-1 px-2 py-2 rounded-lg border transition-colors min-w-fit ${
                isSpeaking 
                  ? 'border-green-500 bg-green-50 dark:bg-green-900/20' 
                  : 'border-gray-300 dark:border-gray-500 bg-white dark:bg-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600'
              }`}
              title={isSpeaking ? "Stop Speaking" : "Speak Last Response"}
            >
              {isSpeaking ? <VolumeX className="w-5 h-5 text-green-500 function-bar-icon" /> : <Volume2 className="w-5 h-5 function-bar-icon" />}
              <span className={`text-xs font-medium ${isSpeaking ? 'text-green-600 dark:text-green-400' : 'text-gray-700 dark:text-gray-200'}`}>
                {isSpeaking ? 'Stop' : 'Speak'}
              </span>
            </button>

            <button
              type="button"
              onClick={handleDualIntegration}
              className={`flex items-center space-x-1 px-2 py-2 rounded-lg border transition-colors min-w-fit ${
                isDualMode 
                  ? 'border-purple-500 bg-gradient-to-r from-purple-50 to-blue-50 dark:from-purple-900/20 dark:to-blue-900/20' 
                  : 'border-gray-300 dark:border-gray-500 bg-white dark:bg-gray-700 hover:bg-gray-100 dark:hover:bg-gray-600'
              }`}
              title={isDualMode ? "Dual Integration Active" : "Activate Dual Integration"}
            >
              <Hammer className={`w-5 h-5 function-bar-icon ${isDualMode ? 'text-purple-600 dark:text-purple-400' : ''}`} />
              <span className={`text-xs font-medium ${isDualMode ? 'text-purple-600 dark:text-purple-400' : 'text-gray-700 dark:text-gray-200'}`}>
                {isDualMode ? 'Dual ON' : 'Dual'}
              </span>
            </button>
          </div>

          {/* File Preview */}
          {showFilePreview && selectedFiles.length > 0 && (
            <div className="mb-3 p-3 bg-gray-50 dark:bg-gray-800 border border-gray-200 dark:border-gray-600 rounded-xl">
              <div className="flex items-center justify-between mb-2">
                <span className="text-sm font-medium text-gray-700 dark:text-gray-300">Selected Files:</span>
                <Button variant="ghost" size="sm" onClick={() => setShowFilePreview(false)}>
                  <X className="w-4 h-4" />
                </Button>
              </div>
              <div className="space-y-2">
                {selectedFiles.map((file, index) => (
                  <div key={index} className="flex items-center justify-between p-2 bg-white dark:bg-gray-700 rounded-lg border">
                    <span className="text-sm text-gray-600 dark:text-gray-300 truncate">{file.name}</span>
                    <Button variant="ghost" size="sm" onClick={() => removeFile(index)}>
                      <X className="w-3 h-3" />
                    </Button>
                  </div>
                ))}
              </div>
              <div className="mt-2 flex space-x-2">
                <Button onClick={uploadSelectedFiles} size="sm">Upload</Button>
                <Button variant="outline" onClick={() => setShowFilePreview(false)} size="sm">Cancel</Button>
              </div>
            </div>
          )}

          {/* Input Bar */}
          <form onSubmit={handleSubmit} className="flex items-center space-x-2">
            <div className="relative flex-1">
              <Input
                value={input}
                onChange={(e) => setInput(e.target.value)}
                placeholder={isConnected ? "Message Forus Heavy API..." : "Connecting..."}
                disabled={!isConnected}
                className="pr-12 py-3 rounded-xl border-gray-200 dark:border-gray-600 bg-white dark:bg-gray-800 text-black dark:text-white"
              />
              <Button
                type="submit"
                size="sm"
                disabled={!input.trim() || !isConnected}
                className="absolute right-2 top-1/2 transform -translate-y-1/2 rounded-lg bg-blue-600 hover:bg-blue-700 text-white"
              >
                <Send className="w-4 h-4" />
              </Button>
            </div>
          </form>

          {/* Connection Status */}
          <div className="flex items-center justify-center mt-2">
            <div className={`flex items-center space-x-2 text-xs ${isConnected ? 'text-green-600 dark:text-green-400' : 'text-red-600 dark:text-red-400'}`}>
              <div className={`w-2 h-2 rounded-full ${isConnected ? 'bg-green-500' : 'bg-red-500'}`}></div>
              <span>{isConnected ? 'Connected' : 'Disconnected'}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### client/src/pages/not-found.tsx
```typescript
import { useLocation } from "wouter";
import { Button } from "@/components/ui/button";

export default function NotFound() {
  const [, setLocation] = useLocation();

  return (
    <div className="min-h-screen bg-white dark:bg-gray-900 flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-6xl font-bold text-gray-900 dark:text-white mb-4">404</h1>
        <p className="text-xl text-gray-600 dark:text-gray-400 mb-8">Page not found</p>
        <Button onClick={() => setLocation("/")}>
          Go Home
        </Button>
      </div>
    </div>
  );
}
```

---

## HOOKS

### client/src/hooks/use-speech.ts
```typescript
import { useState, useRef, useCallback, useEffect } from "react";

interface UseSpeechProps {
  onTranscript?: (transcript: string) => void;
  onSpeakStart?: () => void;
  onSpeakEnd?: () => void;
}

// TypeScript declarations for Web Speech API
interface SpeechRecognitionEvent extends Event {
  results: SpeechRecognitionResultList;
}

interface SpeechRecognitionErrorEvent extends Event {
  error: string;
}

interface SpeechRecognition extends EventTarget {
  continuous: boolean;
  interimResults: boolean;
  lang: string;
  start(): void;
  stop(): void;
  onresult: ((event: SpeechRecognitionEvent) => void) | null;
  onend: (() => void) | null;
  onerror: ((event: SpeechRecognitionErrorEvent) => void) | null;
}

export function useSpeech({ onTranscript, onSpeakStart, onSpeakEnd }: UseSpeechProps = {}) {
  const [isRecording, setIsRecording] = useState(false);
  const [isSupported, setIsSupported] = useState(true);
  const recognitionRef = useRef<SpeechRecognition | null>(null);
  const synthRef = useRef<SpeechSynthesis | null>(null);

  // Initialize speech recognition
  const initializeRecognition = useCallback(() => {
    if (typeof window !== 'undefined') {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      
      if (SpeechRecognition) {
        recognitionRef.current = new SpeechRecognition();
        recognitionRef.current.continuous = false;
        recognitionRef.current.interimResults = false;
        recognitionRef.current.lang = 'en-US';

        recognitionRef.current.onresult = (event: SpeechRecognitionEvent) => {
          const transcript = event.results[0][0].transcript;
          onTranscript?.(transcript);
        };

        recognitionRef.current.onend = () => {
          setIsRecording(false);
        };

        recognitionRef.current.onerror = (event: SpeechRecognitionErrorEvent) => {
          console.error('Speech recognition error:', event.error);
          setIsRecording(false);
        };
      } else {
        setIsSupported(false);
      }
    }
  }, [onTranscript]);

  // Initialize speech synthesis
  const initializeSynthesis = useCallback(() => {
    if (typeof window !== 'undefined' && window.speechSynthesis) {
      synthRef.current = window.speechSynthesis;
    }
  }, []);

  // Initialize on first use
  useEffect(() => {
    initializeRecognition();
    initializeSynthesis();
  }, [initializeRecognition, initializeSynthesis]);

  const startRecording = useCallback(() => {
    if (recognitionRef.current && !isRecording) {
      try {
        recognitionRef.current.start();
        setIsRecording(true);
      } catch (error) {
        console.error('Error starting speech recognition:', error);
      }
    }
  }, [isRecording]);

  const stopRecording = useCallback(() => {
    if (recognitionRef.current && isRecording) {
      recognitionRef.current.stop();
      setIsRecording(false);
    }
  }, [isRecording]);

  // Language detection function
  const detectLanguage = useCallback((text: string): string => {
    // Simple language detection based on character patterns
    const japaneseRegex = /[\u3040-\u309f\u30a0-\u30ff\u4e00-\u9faf]/;
    const chineseRegex = /[\u4e00-\u9fff]/;
    const koreanRegex = /[\uac00-\ud7af]/;
    const arabicRegex = /[\u0600-\u06ff]/;
    const russianRegex = /[\u0400-\u04ff]/;
    const frenchRegex = /[àâäçéèêëïîôùûüÿæœ]/i;
    const spanishRegex = /[ñáéíóúü]/i;
    const germanRegex = /[äöüß]/i;
    const italianRegex = /[àèéìíîòóù]/i;
    const hindiRegex = /[\u0900-\u097F]/;
    const urduRegex = /[\u0600-\u06ff\u0750-\u077f]/; // Urdu uses Arabic script with additional characters
    
    if (japaneseRegex.test(text)) return 'ja-JP';
    if (chineseRegex.test(text)) return 'zh-CN';
    if (koreanRegex.test(text)) return 'ko-KR';
    if (urduRegex.test(text)) return 'ur-PK'; // Urdu (Pakistan)
    if (arabicRegex.test(text)) return 'ar-SA';
    if (russianRegex.test(text)) return 'ru-RU';
    if (hindiRegex.test(text)) return 'hi-IN';
    if (frenchRegex.test(text)) return 'fr-FR';
    if (spanishRegex.test(text)) return 'es-ES';
    if (germanRegex.test(text)) return 'de-DE';
    if (italianRegex.test(text)) return 'it-IT';
    
    return 'en-US'; // Default to English
  }, []);

  const speak = useCallback((text: string) => {
    if (synthRef.current && text) {
      // Stop any ongoing speech
      synthRef.current.cancel();
      
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.rate = 0.9;
      utterance.pitch = 1;
      utterance.volume = 0.8;
      
      // Detect language and set appropriate language and voice
      const detectedLang = detectLanguage(text);
      utterance.lang = detectedLang;
      
      // Try to find a native voice for the detected language
      const voices = synthRef.current.getVoices();
      const nativeVoice = voices.find(voice => 
        voice.lang.startsWith(detectedLang.split('-')[0]) && voice.localService
      ) || voices.find(voice => 
        voice.lang.startsWith(detectedLang.split('-')[0])
      );
      
      if (nativeVoice) {
        utterance.voice = nativeVoice;
      }
      
      utterance.onstart = () => {
        onSpeakStart?.();
      };
      
      utterance.onend = () => {
        onSpeakEnd?.();
      };
      
      utterance.onerror = (event) => {
        console.error('Speech synthesis error:', event.error);
        onSpeakEnd?.();
      };
      
      synthRef.current.speak(utterance);
    }
  }, [onSpeakStart, onSpeakEnd, detectLanguage]);

  const stopSpeaking = useCallback(() => {
    if (synthRef.current) {
      synthRef.current.cancel();
      onSpeakEnd?.();
    }
  }, [onSpeakEnd]);

  return {
    isRecording,
    isSupported,
    startRecording,
    stopRecording,
    speak,
    stopSpeaking
  };
}

// Extend Window interface for TypeScript
declare global {
  interface Window {
    SpeechRecognition: new () => SpeechRecognition;
    webkitSpeechRecognition: new () => SpeechRecognition;
  }
}
```

### client/src/hooks/use-websocket.ts
```typescript
import { useEffect, useRef, useState } from "react";

interface WebSocketMessage {
  type: string;
  content?: string;
  messageType?: 'text' | 'image' | 'file';
  isTyping?: boolean;
}

interface UseWebSocketProps {
  onMessage: (message: WebSocketMessage) => void;
}

export function useWebSocket({ onMessage }: UseWebSocketProps) {
  const [isConnected, setIsConnected] = useState(false);
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<NodeJS.Timeout | null>(null);
  const isUnmountingRef = useRef(false);
  const onMessageRef = useRef(onMessage);
  
  // Keep the callback ref updated
  useEffect(() => {
    onMessageRef.current = onMessage;
  }, [onMessage]);

  useEffect(() => {
    const protocol = window.location.protocol === "https:" ? "wss:" : "ws:";
    const wsUrl = `${protocol}//${window.location.host}/ws`;
    
    const connect = () => {
      // Don't create a new connection if one is already connecting or connected, or if unmounting
      if (isUnmountingRef.current || (wsRef.current && (wsRef.current.readyState === WebSocket.CONNECTING || wsRef.current.readyState === WebSocket.OPEN))) {
        return;
      }

      // Clear any existing reconnect timeout
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
        reconnectTimeoutRef.current = null;
      }

      try {
        wsRef.current = new WebSocket(wsUrl);
        
        wsRef.current.onopen = () => {
          if (!isUnmountingRef.current) {
            setIsConnected(true);
            console.log("WebSocket connected");
          }
        };
        
        wsRef.current.onmessage = (event) => {
          if (!isUnmountingRef.current) {
            try {
              const data = JSON.parse(event.data);
              onMessageRef.current(data);
            } catch (error) {
              console.error("Error parsing WebSocket message:", error);
            }
          }
        };
        
        wsRef.current.onclose = (event) => {
          setIsConnected(false);
          console.log("WebSocket disconnected", event.code, event.reason);
          
          // Only attempt to reconnect if not unmounting and not a clean close
          if (!isUnmountingRef.current && event.code !== 1000) {
            reconnectTimeoutRef.current = setTimeout(() => {
              if (!isUnmountingRef.current) {
                connect();
              }
            }, 3000);
          }
        };
        
        wsRef.current.onerror = (error) => {
          if (!isUnmountingRef.current) {
            console.error("WebSocket error:", error);
            setIsConnected(false);
          }
        };
      } catch (error) {
        if (!isUnmountingRef.current) {
          console.error("Error creating WebSocket connection:", error);
          reconnectTimeoutRef.current = setTimeout(() => {
            if (!isUnmountingRef.current) {
              connect();
            }
          }, 5000);
        }
      }
    };

    connect();

    return () => {
      isUnmountingRef.current = true;
      
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
        reconnectTimeoutRef.current = null;
      }
      
      if (wsRef.current && wsRef.current.readyState === WebSocket.OPEN) {
        wsRef.current.close(1000, "Component unmounting");
      }
    };
  }, []); // Remove onMessage dependency to prevent reconnections

  const sendMessage = (message: any) => {
    if (wsRef.current && wsRef.current.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(message));
    } else {
      console.warn("WebSocket is not connected");
    }
  };

  return { sendMessage, isConnected };
}
```

### client/src/hooks/use-toast.ts
```typescript
import { useState, useCallback } from "react";

interface Toast {
  id: string;
  title?: string;
  description?: string;
  variant?: "default" | "destructive";
  duration?: number;
}

export function useToast() {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const toast = useCallback(
    ({ title, description, variant = "default", duration = 5000 }: Omit<Toast, "id">) => {
      const id = Math.random().toString(36).substr(2, 9);
      const newToast: Toast = { id, title, description, variant, duration };

      setToasts((prevToasts) => [...prevToasts, newToast]);

      if (duration > 0) {
        setTimeout(() => {
          setToasts((prevToasts) => prevToasts.filter((t) => t.id !== id));
        }, duration);
      }

      return id;
    },
    []
  );

  const dismiss = useCallback((id: string) => {
    setToasts((prevToasts) => prevToasts.filter((t) => t.id !== id));
  }, []);

  return {
    toast,
    dismiss,
    toasts,
  };
}
```

### client/src/hooks/use-mobile.tsx
```typescript
import { useState, useEffect } from "react";

const MOBILE_BREAKPOINT = 768;

export function useIsMobile() {
  const [isMobile, setIsMobile] = useState<boolean | undefined>(undefined);

  useEffect(() => {
    const mql = window.matchMedia(`(max-width: ${MOBILE_BREAKPOINT - 1}px)`);
    const onChange = () => {
      setIsMobile(window.innerWidth < MOBILE_BREAKPOINT);
    };
    mql.addEventListener("change", onChange);
    setIsMobile(window.innerWidth < MOBILE_BREAKPOINT);
    return () => mql.removeEventListener("change", onChange);
  }, []);

  return !!isMobile;
}
```

---

## COMPONENTS

### client/src/components/theme-provider.tsx
```typescript
import { createContext, useContext, useEffect, useState } from "react";

type Theme = "dark" | "light" | "system";

type ThemeProviderProps = {
  children: React.ReactNode;
  defaultTheme?: Theme;
  storageKey?: string;
};

type ThemeProviderState = {
  theme: Theme;
  setTheme: (theme: Theme) => void;
};

const initialState: ThemeProviderState = {
  theme: "system",
  setTheme: () => null,
};

const ThemeProviderContext = createContext<ThemeProviderState>(initialState);

export function ThemeProvider({
  children,
  defaultTheme = "light",
  storageKey = "vite-ui-theme",
  ...props
}: ThemeProviderProps) {
  const [theme, setTheme] = useState<Theme>(
    () => (localStorage.getItem(storageKey) as Theme) || defaultTheme
  );

  useEffect(() => {
    const root = window.document.documentElement;
    
    root.classList.remove("light", "dark");

    if (theme === "system") {
      const systemTheme = window.matchMedia("(prefers-color-scheme: dark)")
        .matches
        ? "dark"
        : "light";

      root.classList.add(systemTheme);
      console.log("Applied system theme:", systemTheme);
      return;
    }

    root.classList.add(theme);
    console.log("Applied theme:", theme);
  }, [theme]);

  const value = {
    theme,
    setTheme: (theme: Theme) => {
      localStorage.setItem(storageKey, theme);
      setTheme(theme);
    },
  };

  return (
    <ThemeProviderContext.Provider {...props} value={value}>
      {children}
    </ThemeProviderContext.Provider>
  );
}

export const useTheme = () => {
  const context = useContext(ThemeProviderContext);

  if (context === undefined)
    throw new Error("useTheme must be used within a ThemeProvider");

  return context;
};
```

### client/src/components/theme-toggle.tsx
```typescript
import { Moon, Sun, Monitor } from "lucide-react";
import { Button } from "@/components/ui/button";
import { useTheme } from "./theme-provider";

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  const toggleTheme = () => {
    if (theme === "light") {
      setTheme("dark");
    } else if (theme === "dark") {
      setTheme("system");
    } else {
      setTheme("light");
    }
  };

  const getIcon = () => {
    switch (theme) {
      case "light":
        return <Sun className="h-4 w-4" />;
      case "dark":
        return <Moon className="h-4 w-4" />;
      case "system":
        return <Monitor className="h-4 w-4" />;
      default:
        return <Sun className="h-4 w-4" />;
    }
  };

  return (
    <Button variant="ghost" size="sm" onClick={toggleTheme}>
      {getIcon()}
    </Button>
  );
}
```

---

## LIBRARY FILES

### client/src/lib/utils.ts
```typescript
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### client/src/lib/queryClient.ts
```typescript
import { QueryClient } from "@tanstack/react-query";

const defaultFetcher = async (url: string, options?: RequestInit) => {
  const response = await fetch(url, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  return response.json();
};

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: ({ queryKey }) => defaultFetcher(queryKey[0] as string),
      staleTime: 1000 * 60 * 5, // 5 minutes
      retry: (failureCount, error) => {
        // Don't retry on 4xx errors
        if (error instanceof Error && error.message.includes("4")) {
          return false;
        }
        // Retry up to 3 times for other errors
        return failureCount < 3;
      },
    },
  },
});

// Helper function for mutations
export const apiRequest = async (url: string, options?: RequestInit) => {
  return defaultFetcher(url, options);
};
```

---

## SHARED SCHEMA

### shared/schema.ts
```typescript
import { sql } from "drizzle-orm";
import { pgTable, text, varchar, timestamp, jsonb } from "drizzle-orm/pg-core";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";

export const users = pgTable("users", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  username: text("username").notNull().unique(),
  password: text("password").notNull(),
  createdAt: timestamp("created_at").defaultNow(),
});

export const conversations = pgTable("conversations", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  userId: varchar("user_id"),
  title: text("title"),
  createdAt: timestamp("created_at").defaultNow(),
  updatedAt: timestamp("updated_at").defaultNow(),
});

export const messages = pgTable("messages", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  conversationId: varchar("conversation_id").notNull(),
  content: text("content").notNull(),
  isUser: text("is_user").notNull(), // 'true' or 'false' as text
  messageType: text("message_type").default("text"), // 'text', 'image', 'file'
  metadata: jsonb("metadata"), // For storing additional data like file info, image URLs, etc.
  createdAt: timestamp("created_at").defaultNow(),
});

export const insertUserSchema = createInsertSchema(users).omit({
  id: true,
  createdAt: true,
});

export const insertConversationSchema = createInsertSchema(conversations).omit({
  id: true,
  createdAt: true,
  updatedAt: true,
});

export const insertMessageSchema = createInsertSchema(messages).omit({
  id: true,
  createdAt: true,
});

export type InsertUser = z.infer<typeof insertUserSchema>;
export type User = typeof users.$inferSelect;

export type InsertConversation = z.infer<typeof insertConversationSchema>;
export type Conversation = typeof conversations.$inferSelect;

export type InsertMessage = z.infer<typeof insertMessageSchema>;
export type Message = typeof messages.$inferSelect;

// API message format for WebSocket communication
export const apiMessageSchema = z.object({
  content: z.string(),
  type: z.enum(['text', 'image', 'file']).default('text'),
  timestamp: z.string().optional(),
});

export type ApiMessage = z.infer<typeof apiMessageSchema>;
```

---

## CSS STYLES

### client/src/index.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 240 10% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 240 10% 3.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 240 10% 3.9%;
    --primary: 240 9% 10%;
    --primary-foreground: 0 0% 98%;
    --secondary: 240 4.8% 95.9%;
    --secondary-foreground: 240 5.9% 10%;
    --muted: 240 4.8% 95.9%;
    --muted-foreground: 240 3.8% 45%;
    --accent: 240 4.8% 95.9%;
    --accent-foreground: 240 5.9% 10%;
    --destructive: 0 72% 51%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 5.9% 90%;
    --input: 240 5.9% 90%;
    --ring: 240 5.9% 10%;
    --chart-1: 173 58% 39%;
    --chart-2: 12 76% 61%;
    --chart-3: 197 37% 24%;
    --chart-4: 43 74% 66%;
    --chart-5: 27 87% 67%;
    --radius: 0.5rem;
    --sidebar-background: 0 0% 98%;
    --sidebar-foreground: 240 5.3% 26.1%;
    --sidebar-primary: 240 9% 10%;
    --sidebar-primary-foreground: 0 0% 98%;
    --sidebar-accent: 240 4.8% 95.9%;
    --sidebar-accent-foreground: 240 5.9% 10%;
    --sidebar-border: 220 13% 91%;
    --sidebar-ring: 217.2 10.6% 64.9%;
  }

  .dark {
    --background: 240 10% 3.9%;
    --foreground: 0 0% 98%;
    --card: 240 10% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 240 10% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 0% 98%;
    --primary-foreground: 240 5.9% 10%;
    --secondary: 240 3.7% 15.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 240 3.7% 15.9%;
    --muted-foreground: 240 5% 64.9%;
    --accent: 240 3.7% 15.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 72% 51%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 240 4.9% 83.9%;
    --chart-1: 220 70% 50%;
    --chart-2: 160 60% 45%;
    --chart-3: 30 80% 55%;
    --chart-4: 280 65% 60%;
    --chart-5: 340 75% 55%;
    --sidebar-background: 240 5.9% 10%;
    --sidebar-foreground: 240 4.8% 95.9%;
    --sidebar-primary: 224.3 76.3% 94.1%;
    --sidebar-primary-foreground: 220.9 39.3% 11%;
    --sidebar-accent: 240 3.7% 15.9%;
    --sidebar-accent-foreground: 240 4.8% 95.9%;
    --sidebar-border: 240 3.7% 15.9%;
    --sidebar-ring: 217.2 10.6% 64.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}

/* Custom styles for landing page */
.grid-background {
  background-image: 
    linear-gradient(to right, #f0f0f0 1px, transparent 1px),
    linear-gradient(to bottom, #f0f0f0 1px, transparent 1px);
  background-size: 40px 40px;
}

.grid-overlay {
  background: radial-gradient(circle at center, transparent 0%, rgba(255,255,255,0.8) 100%);
}

/* Responsive typography */
.title-responsive {
  font-size: 2.5rem;
}

.tagline-responsive {
  font-size: 1rem;
}

@media (min-width: 640px) {
  .title-responsive {
    font-size: 3.5rem;
  }
  .tagline-responsive {
    font-size: 1.125rem;
  }
}

@media (min-width: 1024px) {
  .title-responsive {
    font-size: 4rem;
  }
  .tagline-responsive {
    font-size: 1.25rem;
  }
}

/* Typing animation */
@keyframes typing-dot {
  0%, 20% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.5);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.typing-dot-1 {
  animation: typing-dot 1.4s infinite ease-in-out;
  animation-delay: 0s;
}

.typing-dot-2 {
  animation: typing-dot 1.4s infinite ease-in-out;
  animation-delay: 0.2s;
}

.typing-dot-3 {
  animation: typing-dot 1.4s infinite ease-in-out;
  animation-delay: 0.4s;
}

/* Mobile responsive function bar */
.function-bar-mobile {
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.function-bar-mobile::-webkit-scrollbar {
  display: none;
}

/* Function bar icon sizing */
.function-bar-icon {
  width: 1rem;
  height: 1rem;
}

@media (min-width: 768px) {
  .function-bar-icon {
    width: 1.25rem;
    height: 1.25rem;
  }
}

@media (min-width: 1024px) {
  .function-bar-icon {
    width: 1.5rem;
    height: 1.5rem;
  }
}

/* Force light theme on message bar */
.light .message-bar {
  background-color: white !important;
  border-color: #d1d5db !important;
}

/* Force visibility of typing dots */
.typing-dot-1,
.typing-dot-2,
.typing-dot-3 {
  background-color: #6b7280 !important;
}

.dark .typing-dot-1,
.dark .typing-dot-2,
.dark .typing-dot-3 {
  background-color: #9ca3af !important;
}
```

---

## ENVIRONMENT SETUP

### .replit
```
modules = ["nodejs-20", "python-3.11"]
run = "npm run dev"

[nix]
channel = "stable-24_05"

[unitTest]
language = "nodejs"

[deployment]
run = ["sh", "-c", "npm run start"]
deploymentTarget = "cloudrun"

[[ports]]
localPort = 5000
externalPort = 80

[[ports]]
localPort = 5173
externalPort = 8080
```

---

## GETTING STARTED

1. **Environment Variables**: Set up your OpenRouter API keys:
   ```
   OPENROUTER_API_KEY_1=your_primary_key_here
   OPENROUTER_API_KEY_2=your_secondary_key_here
   ```

2. **Install Dependencies**:
   ```bash
   npm install
   ```

3. **Start Development Server**:
   ```bash
   npm run dev
   ```

4. **Build for Production**:
   ```bash
   npm run build
   npm start
   ```

---

Your **Forus Heavy API** is now complete with dual API integration, multi-language speech support including Urdu, real-time chat, and a modern responsive interface!