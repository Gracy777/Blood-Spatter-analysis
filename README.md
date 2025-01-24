import { useState, useRef, useEffect } from 'react'
import { Button } from "/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "/components/ui/card"
import { Upload, Camera, Check } from "lucide-react"
import { Star } from "lucide-react"
import { Label } from "/components/ui/label"
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "/components/ui/table"

export default function BloodSpatterAnalyzer() {
  const [imageSrc, setImageSrc] = useState<string | null>(null)
  const [bloodDrops, setBloodDrops] = useState<{ x: number, y: number }[]>([])
  const [selectedDrop, setSelectedDrop] = useState<number | null>(null)
  const [distances, setDistances] = useState<{ index1: number, index2: number, distance: number }[]>([])
  const videoRef = useRef<HTMLVideoElement>(null)
  const canvasRef = useRef<HTMLCanvasElement>(null)

  const handleImageUpload = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0]
    if (file) {
      const reader = new FileReader()
      reader.onloadend = () => {
        setImageSrc(reader.result as string)
      }
      reader.readAsDataURL(file)
    }
  }

  const handleImageClick = (event: React.MouseEvent<HTMLImageElement>) => {
    if (!imageSrc) return

    const rect = event.currentTarget.getBoundingClientRect()
    const x = event.clientX - rect.left
    const y = event.clientY - rect.top

    setBloodDrops([...bloodDrops, { x, y }])
  }

  const handleDropClick = (index: number) => {
    if (selectedDrop === null) {
      setSelectedDrop(index)
    } else {
      const drop1 = bloodDrops[selectedDrop]
      const drop2 = bloodDrops[index]
      const dist = Math.sqrt((drop2.x - drop1.x) ** 2 + (drop2.y - drop1.y) ** 2)
      setDistances([...distances, { index1: selectedDrop, index2: index, distance: dist }])
      setSelectedDrop(null)
    }
  }

  const startCamera = async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true })
      if (videoRef.current) {
        videoRef.current.srcObject = stream
      }
    } catch (error) {
      console.error("Error accessing camera:", error)
    }
  }

  const captureImage = () => {
    if (videoRef.current && canvasRef.current) {
      const video = videoRef.current
      const canvas = canvasRef.current
      const context = canvas.getContext('2d')
      if (context) {
        canvas.width = video.videoWidth
        canvas.height = video.videoHeight
        context.drawImage(video, 0, 0, canvas.width, canvas.height)
        setImageSrc(canvas.toDataURL('image/png'))
      }
    }
  }

  useEffect(() => {
    startCamera()
    return () => {
      if (videoRef.current) {
        const stream = videoRef.current.srcObject
        if (stream instanceof MediaStream) {
          const tracks = stream.getTracks()
          tracks.forEach(track => track.stop())
        }
      }
    }
  }, [])

  return (
    <Card className="w-full max-w-4xl mx-auto">
      <CardHeader>
        <CardTitle className="text-2xl font-bold">Blood Spatter Analyzer</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="flex items-center justify-center">
          <label className="flex flex-col items-center justify-center w-full h-64 border-2 border-gray-300 border-dashed rounded-lg cursor-pointer bg-gray-50 hover:bg-gray-100">
            <div className="flex flex-col items-center justify-center pt-5 pb-6">
              <Upload className="w-10 h-10 text-gray-400" />
              <p className="mb-2 text-sm text-gray-500"><span className="font-semibold">Click to upload</span> or drag and drop</p>
              <p className="text-xs text-gray-500">PNG, JPG, GIF up to 10MB</p>
            </div>
            <input type="file" className="hidden" accept="image/*" onChange={handleImageUpload} />
          </label>
        </div>
        <div className="flex items-center justify-center">
          <video ref={videoRef} autoPlay playsInline className="w-full h-auto max-h-64" />
          <Button onClick={captureImage} className="ml-4">
            <Camera className="mr-2" />
            Capture
          </Button>
        </div>
        <canvas ref={canvasRef} className="hidden" />
        {imageSrc && (
          <div className="relative">
            <img src={imageSrc} alt="Blood Spatter" className="w-full h-auto" onClick={handleImageClick} />
            {bloodDrops.map((drop, index) => (
              <div
                key={index}
                className="absolute w-4 h-4 bg-red-500 rounded-full cursor-pointer"
                style={{ left: `${drop.x}px`, top: `${drop.y}px` }}
                onClick={() => handleDropClick(index)}
              >
                {selectedDrop === index && <Star className="absolute w-4 h-4 text-white" />}
              </div>
            ))}
          </div>
        )}
        {distances.length > 0 && (
          <div className="mt-4">
            <Table>
              <TableHeader>
                <TableRow>
                  <TableHead>Drop 1</TableHead>
                  <TableHead>Drop 2</TableHead>
                  <TableHead>Distance (px)</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                {distances.map((dist, index) => (
                  <TableRow key={index}>
                    <TableCell>{dist.index1 + 1}</TableCell>
                    <TableCell>{dist.index2 + 1}</TableCell>
                    <TableCell>{dist.distance.toFixed(2)}</TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </div>
        )}
        <div className="mt-4">
          <Label className="text-sm text-gray-500">Instructions:</Label>
          <ul className="list-disc list-inside text-sm text-gray-500">
            <li>Upload an image or capture one using the camera.</li>
            <li>Click on the image to mark blood drops.</li>
            <li>Select two marked blood drops to measure the distance between them.</li>
          </ul>
        </div>
      </CardContent>
    </Card>
  )
}
