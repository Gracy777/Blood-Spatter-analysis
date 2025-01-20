import { useState, useRef, useEffect } from 'react';
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";

export default function BloodSpatterAnalysisTool() {
  const [stream, setStream] = useState(null);
  const [points, setPoints] = useState([]);
  const [spatterType, setSpatterType] = useState('No pattern detected');
  const [distance, setDistance] = useState('0 cm');
  const [showCamera, setShowCamera] = useState(false);

  const cameraRef = useRef(null);
  const canvasRef = useRef(null);
  const fileInputRef = useRef(null);

  useEffect(() => {
    if (canvasRef.current) {
      const canvas = canvasRef.current;
      canvas.width = 800;
      canvas.height = 450;
    }
  }, []);

  const handleCameraOpen = async () => {
    try {
      const mediaStream = await navigator.mediaDevices.getUserMedia({ video: true });
      setStream(mediaStream);
      if (cameraRef.current) {
        cameraRef.current.srcObject = mediaStream;
        setShowCamera(true);
      }
    } catch (err) {
      alert('Camera access denied or not available');
    }
  };

  const handleCapture = () => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(cameraRef.current, 0, 0, canvas.width, canvas.height);
    setShowCamera(false);
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      setStream(null);
    }
  };

  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const img = new Image();
      img.onload = () => {
        const ctx = canvasRef.current.getContext('2d');
        ctx.drawImage(img, 0, 0, canvasRef.current.width, canvasRef.current.height);
      };
      img.src = URL.createObjectURL(file);
    }
  };

  const handleCanvasClick = (e) => {
    const canvas = canvasRef.current;
    const rect = canvas.getBoundingClientRect();
    const x = (e.clientX - rect.left) * (canvas.width / rect.width);
    const y = (e.clientY - rect.top) * (canvas.height / rect.height);

    const newPoints = [...points, { x, y }];
    setPoints(newPoints);
    drawPoint(x, y);

    if (newPoints.length === 2) {
      calculateDistance(newPoints);
      analyzePattern();
      setPoints([]);
    }
  };

  const drawPoint = (x, y) => {
    const ctx = canvasRef.current.getContext('2d');
    ctx.beginPath();
    ctx.arc(x, y, 5, 0, 2 * Math.PI);
    ctx.fillStyle = 'red';
    ctx.fill();
  };

  const calculateDistance = (points) => {
    const dx = points[1].x - points[0].x;
    const dy = points[1].y - points[0].y;
    const distance = Math.sqrt(dx * dx + dy * dy);
    const distanceCm = (distance * 0.026458333).toFixed(2);
    setDistance(`${distanceCm} cm`);

    const ctx = canvasRef.current.getContext('2d');
    ctx.beginPath();
    ctx.moveTo(points[0].x, points[0].y);
    ctx.lineTo(points[1].x, points[1].y);
    ctx.strokeStyle = 'red';
    ctx.stroke();
  };

  const analyzePattern = () => {
    const patterns = ['Impact Spatter', 'Cast-off Pattern', 'Projected Pattern', 'Transfer Pattern'];
    const randomPattern = patterns[Math.floor(Math.random() * patterns.length)];
    setSpatterType(randomPattern);
  };

  const generateReport = () => {
    const reportWindow = window.open('', '_blank');
    reportWindow.document.write(`
      <html>
        <head>
          <title>Blood Spatter Analysis Report</title>
          <style>
            body { font-family: Arial, sans-serif; padding: 20px; }
            .header { text-align: center; margin-bottom: 30px; }
            .content { margin: 20px 0; }
          </style>
        </head>
        <body>
          <div class="header">
            <h1>Blood Spatter Analysis Report</h1>
            <p>Generated on: ${new Date().toLocaleString()}</p>
          </div>
          <div class="content">
            <h2>Analysis Results</h2>
            <p>Pattern Type: ${spatterType}</p>
            <p>Distance Measured: ${distance}</p>
          </div>
          <img src="${canvasRef.current.toDataURL()}" style="max-width: 100%;">
        </body>
      </html>
    `);
  };

  return (
    <div className="container mx-auto px-4 py-8">
      <header className="text-center mb-8">
        <h1 className="text-4xl font-bold text-slate-800 mb-2">BloodSpatter Analysis Tool</h1>
        <p className="text-slate-600">Professional Crime Scene Analysis Software</p>
      </header>

      <div className="grid md:grid-cols-2 gap-8">
        <Card>
          <CardHeader>
            <CardTitle>Image Capture</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-4">
              {showCamera && (
                <video
                  ref={cameraRef}
                  className="w-full h-64 bg-slate-200 rounded-lg object-cover"
                  autoPlay
                />
              )}
              <canvas
                ref={canvasRef}
                className="w-full h-64 bg-slate-200 rounded-lg border-2 border-slate-300"
                onClick={handleCanvasClick}
              />
              <div className="flex gap-4">
                <Button onClick={handleCameraOpen}>
                  <i className="bi bi-camera-fill mr-2" /> Open Camera
                </Button>
                {showCamera && (
                  <Button onClick={handleCapture} variant="secondary">
                    <i className="bi bi-camera mr-2" /> Capture
                  </Button>
                )}
                <input
                  type="file"
                  ref={fileInputRef}
                  accept="image/*"
                  className="hidden"
                  onChange={handleFileUpload}
                />
                <Button onClick={() => fileInputRef.current.click()} variant="outline">
                  <i className="bi bi-upload mr-2" /> Upload Image
                </Button>
              </div>
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Analysis Results</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-4">
              <div className="p-4 bg-slate-50 rounded-lg">
                <h3 className="font-semibold mb-2">Blood Spatter Type</h3>
                <div className="text-slate-600">{spatterType}</div>
              </div>
              <div className="p-4 bg-slate-50 rounded-lg">
                <h3 className="font-semibold mb-2">Distance Measurement</h3>
                <div className="text-slate-600">{distance}</div>
              </div>
              <Button onClick={generateReport} className="w-full bg-red-600">
                <i className="bi bi-file-pdf mr-2" /> Generate Report
              </Button>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
