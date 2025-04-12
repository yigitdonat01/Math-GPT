# Math-GPT
import 'package:flutter/material.dart'; import 'package:google_ml_kit/google_ml_kit.dart'; import 'package:image_picker/image_picker.dart'; import 'dart:io';

void main() { runApp(const MathGPTApp()); }

class MathGPTApp extends StatelessWidget { const MathGPTApp({super.key});

@override Widget build(BuildContext context) { return MaterialApp( title: 'Math GPT', theme: ThemeData(primarySwatch: Colors.blueGrey), home: const HomePage(), debugShowCheckedModeBanner: false, ); } }

class HomePage extends StatelessWidget { const HomePage({super.key});

@override Widget build(BuildContext context) { final List<Map<String, dynamic>> buttons = [ { "title": "Fotoğrafla Soru Çöz", "icon": Icons.camera_alt, "page": const OcrSolverPage() }, { "title": "Konu Anlatımı", "icon": Icons.menu_book, "page": const TopicDetailPage(title: "1. Dereceden Denklemler") }, ];

return Scaffold( appBar: AppBar(title: const Text("Math GPT")), body: GridView.count( crossAxisCount: 2, padding: const EdgeInsets.all(16), children: buttons.map((btn) { return Card( shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)), elevation: 4, child: InkWell( onTap: () => Navigator.push( context, MaterialPageRoute(builder: (_) => btn["page"]), ), child: Center( child: Column( mainAxisSize: MainAxisSize.min, children: [ Icon(btn["icon"], size: 40, color: Colors.blueGrey), const SizedBox(height: 10), Text(btn["title"], textAlign: TextAlign.center), ], ), ), ), ); }).toList(), ), ); 

} }

class TopicDetailPage extends StatelessWidget { final String title; const TopicDetailPage({super.key, required this.title});

@override Widget build(BuildContext context) { return Scaffold( appBar: AppBar(title: Text(title)), body: ListView( padding: const EdgeInsets.all(16), children: const [ Text("Tanım", style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)), SizedBox(height: 8), Text("1. dereceden denklem, bilinmeyenin üssünün 1 olduğu denklemdir. Genel formu ax + b = 0 şeklindedir."), SizedBox(height: 16), Text("Formül", style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)), Text("ax + b = 0 → x = -b / a"), SizedBox(height: 16), Text("Örnek", style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)), Text("2x + 4 = 0 → x = -2"), ], ), ); } }

class OcrSolverPage extends StatefulWidget { const OcrSolverPage({super.key});

@override State createState() => _OcrSolverPageState(); }

class _OcrSolverPageState extends State { String result = ""; File? image;

Future pickImage() async { final picked = await ImagePicker().pickImage(source: ImageSource.camera); if (picked == null) return;

image = File(picked.path); final inputImage = InputImage.fromFile(image!); final recognizer = GoogleMlKit.vision.textRecognizer(); final RecognizedText recognizedText = await recognizer.processImage(inputImage); recognizer.close(); final scanned = recognizedText.text.replaceAll(' ', ''); setState(() { result = _solveEquation(scanned); }); 

}

String _solveEquation(String eq) { try { final parts = eq.split('='); if (parts.length != 2) return "Denklem tanınamadı";

final left = parts[0]; final right = parts[1]; final a = RegExp(r'([\-]?\d*)x').firstMatch(left)?.group(1); final b = RegExp(r'([+-]?\d+)(?!x)').firstMatch(left)?.group(1); final c = int.tryParse(right); if (a == null || b == null || c == null) return "Denklem okunamadı"; final aVal = a.isEmpty || a == '+' ? 1 : a == '-' ? -1 : int.parse(a); final bVal = int.parse(b); final x = (c - bVal) / aVal; return "Çözüm: x = $x"; } catch (e) { return "Hata: \${e.toString()}"; } 

}

@override Widget build(BuildContext context) { return Scaffold( appBar: AppBar(title: const Text("Fotoğrafla Soru Çöz")), body: Padding( padding: const EdgeInsets.all(16), child: Column( children: [ ElevatedButton(onPressed: pickImage, child: const Text("Fotoğraf Çek")), const SizedBox(height: 20), if (image != null) Image.file(image!, height: 200), const SizedBox(height: 20), Text(result), ], ), ), ); } }

