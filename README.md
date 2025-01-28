// Install dependencies first using: npm install sharp fabric caman pixi.js
const sharp = require('sharp');
const { fabric } = require('fabric');
const Caman = require('caman').Caman;
const PIXI = require('pixi.js');

// 1. Image Crop and Resize
sharp('input-image.jpg')
  .resize(800, 600)
  .toFile('output-resize.jpg', (err, info) => {
    if (err) throw err;
    console.log('Resize Info:', info);
  });

// 2. Rotate and Flip
sharp('input-image.jpg')
  .rotate(90) // Rotate by 90°
  .flip() // Flip vertically
  .toFile('output-rotate-flip.jpg', (err, info) => {
    if (err) throw err;
    console.log('Rotate & Flip Info:', info);
  });

// 3. Adjust Brightness, Contrast, and Saturation
sharp('input-image.jpg')
  .modulate({
    brightness: 1.2, // Increase brightness by 20%
    saturation: 1.5, // Increase saturation by 50%
    contrast: 1.5, // Increase contrast by 50%
  })
  .toFile('output-adjust.jpg', (err, info) => {
    if (err) throw err;
    console.log('Adjustment Info:', info);
  });

// 4. Apply Filters (Grayscale and Sepia)
Caman('input-image.jpg', function () {
  this.greyscale() // Convert to grayscale
    .sepia() // Apply sepia filter
    .render(function () {
      this.save('output-filters.jpg');
      console.log('Filters applied!');
    });
});

// 5. Add Text Overlays using Fabric.js
let canvas = new fabric.Canvas(null, { width: 800, height: 600 }); // No actual DOM element needed

fabric.Image.fromURL('input-image.jpg', (img) => {
  canvas.add(img);
  let text = new fabric.Text('Hello, World!', {
    left: 100,
    top: 100,
    fontSize: 24,
    fill: 'white',
  });
  canvas.add(text);
  canvas.renderAll();

  const dataURL = canvas.toDataURL(); // Export canvas to an image
  console.log('Canvas Text Added: Data URL Created');
});

// 6. Draw on Images
canvas.isDrawingMode = true;
canvas.freeDrawingBrush.color = 'red';
canvas.freeDrawingBrush.width = 5;
console.log('Drawing Mode Enabled.');

// 7. Sharpen Images
sharp('input-image.jpg')
  .sharpen()
  .toFile('output-sharpen.jpg', (err, info) => {
    if (err) throw err;
    console.log('Sharpen Info:', info);
  });

// 8. Color Adjustment (Hue, Saturation, Lightness)
sharp('input-image.jpg')
  .modulate({
    hue: 180, // Adjust hue by 180°
    saturation: 1.2, // Increase saturation by 20%
    lightness: 0.8, // Decrease lightness by 20%
  })
  .toFile('output-color-adjust.jpg', (err, info) => {
    if (err) throw err;
    console.log('Color Adjustment Info:', info);
  });

// 9. Add Borders or Frames
sharp('input-image.jpg')
  .extend({
    top: 10,
    bottom: 10,
    left: 10,
    right: 10,
    background: { r: 255, g: 255, b: 255 }, // White border
  })
  .toFile('output-border.jpg', (err, info) => {
    if (err) throw err;
    console.log('Border Added Info:', info);
  });

// 10. Sticker and Emoji Placement using Pixi.js
let app = new PIXI.Application({ width: 800, height: 600 });
document.body.appendChild(app.view);

PIXI.Loader.shared.add('emoji', 'emoji.png').load(function (loader, resources) {
  let emoji = new PIXI.Sprite(resources.emoji.texture);
  emoji.x = 100;
  emoji.y = 100;
  app.stage.addChild(emoji);
  console.log('Emoji Added.');
});

import React, { useState } from 'react';
import {
  Modal,
  TextInput,
  Button,
  Text,
  View,
  StyleSheet,
} from 'react-native';

export default function App() {
  const [passcode, setPasscode] = useState('');
  const [accessGranted, setAccessGranted] = useState(false);
  const [modalVisible, setModalVisible] = useState(true);

  // Passcode validation
  const checkPasscode = () => {
    if (passcode === '6988') {
      setAccessGranted(true);
    } else {
      alert('Incorrect passcode');
    }
  };

  return (
    <View style={styles.container}>
      {accessGranted ? (
        // Show terms and conditions content when passcode is correct
        <View>
          <Text style={styles.header}>Terms and Conditions</Text>
          <Text style={styles.content}>
            These are the terms and conditions. You can update this text freely
            as needed. Ensure compliance with legal requirements.
          </Text>
        </View>
      ) : (
        // Passcode input section
        <View style={styles.passcodeContainer}>
          <TextInput
            style={styles.input}
            placeholder="Enter passcode"
            value={passcode}
            onChangeText={setPasscode}
            secureTextEntry
          />
          <Button title="Submit" onPress={checkPasscode} />
        </View>
      )}

      {/* Modal Pop-up for Terms and Conditions */}
      <Modal
        transparent={true}
        visible={modalVisible}
        animationType="slide"
        onRequestClose={() => setModalVisible(false)}
      >
        <View style={styles.modalOverlay}>
          <View style={styles.modalContent}>
            <Text style={styles.modalHeader}>Terms and Conditions</Text>
            <Text style={styles.modalBody}>
              By using this app, you agree to our terms and conditions.
            </Text>
            <Button title="I Agree" onPress={() => setModalVisible(false)} />
          </View>
        </View>
      </Modal>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#2c3e50',
    justifyContent: 'center',
    alignItems: 'center',
  },
  passcodeContainer: {
    alignItems: 'center',
  },
  input: {
    borderWidth: 1,
    borderColor: '#fff',
    backgroundColor: '#fff',
    padding: 10,
    marginBottom: 10,
    borderRadius: 5,
    width: 200,
  },
  header: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#fff',
    marginBottom: 10,
  },
  content: {
    fontSize: 16,
    color: '#fff',
    textAlign: 'center',
    paddingHorizontal: 20,
  },
  modalOverlay: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(0, 0, 0, 0.5)',
  },
  modalContent: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    alignItems: 'center',
  },
  modalHeader: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  modalBody: {
    fontSize: 14,
    textAlign: 'center',
    marginBottom: 20,
  },
});
