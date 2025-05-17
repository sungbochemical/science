# 1<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>아미노산 조합 학습</title>
    <style>
        body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; }
        .button-container { margin-bottom: 20px; }
        button { padding: 10px 20px; margin: 5px; cursor: pointer; }
        #result-container { margin-top: 20px; font-size: 1.2em; font-weight: bold; }
        #structure-container { width: 300px; height: 300px; border: 1px solid #ccc; margin-top: 20px; }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://threejs.org/examples/js/controls/OrbitControls.js"></script>
</head>
<body>
    <h1>아미노산 조합 학습</h1>

    <div class="button-container">
        <button onclick="appendSequence('G')">G</button>
        <button onclick="appendSequence('C')">C</button>
        <button onclick="appendSequence('A')">A</button>
        <button onclick="appendSequence('U')">U</button>
    </div>

    <div>
        조합된 염기 서열: <span id="sequence"></span>
    </div>

    <div class="button-container">
        <button onclick="checkAminoAcid()">결과 확인</button>
        <button onclick="resetSequence()">초기화</button>
    </div>

    <div id="result-container"></div>

    <div id="structure-container"></div>

    <script>
        const sequenceDisplay = document.getElementById('sequence');
        const resultDisplay = document.getElementById('result-container');
        const structureContainer = document.getElementById('structure-container');
        let currentSequence = '';

        const aminoAcidMap = {
            'GGG': '글라이신 (Glycine)',
            'GGA': '글라이신 (Glycine)',
            'GGU': '글라이신 (Glycine)',
            'GGC': '글라이신 (Glycine)',
            'GCU': '알라닌 (Alanine)',
            'GCC': '알라닌 (Alanine)',
            'GCA': '알라닌 (Alanine)',
            'GCG': '알라닌 (Alanine)',
            'GUU': '발린 (Valine)',
            'GUC': '발린 (Valine)',
            'GUA': '발린 (Valine)',
            'GUG': '발린 (Valine)',
            'UUA': '류신 (Leucine)',
            'UUG': '류신 (Leucine)',
            'CUU': '류신 (Leucine)',
            'CUC': '류신 (Leucine)',
            'CUA': '류신 (Leucine)',
            'CUG': '류신 (Leucine)',
            'AUU': '아이소류신 (Isoleucine)',
            'AUC': '아이소류신 (Isoleucine)',
            'AUA': '아이소류신 (Isoleucine)',
            'AUG': '메티오닌 (Methionine)',
            'UUU': '페닐알라닌 (Phenylalanine)',
            'UUC': '페닐알라닌 (Phenylalanine)',
            'UAU': '티로신 (Tyrosine)',
            'UAC': '티로신 (Tyrosine)',
            'UGG': '트립토판 (Tryptophan)',
            'UCU': '세린 (Serine)',
            'UCC': '세린 (Serine)',
            'UCA': '세린 (Serine)',
            'UCG': '세린 (Serine)',
            'AGU': '세린 (Serine)',
            'AGC': '세린 (Serine)',
            'ACU': '트레오닌 (Threonine)',
            'ACC': '트레오닌 (Threonine)',
            'ACA': '트레오닌 (Threonine)',
            'ACG': '트레오닌 (Threonine)',
            'UGU': '시스테인 (Cysteine)',
            'UGC': '시스테인 (Cysteine)',
            'CCU': '프롤린 (Proline)',
            'CCC': '프롤린 (Proline)',
            'CCA': '프롤린 (Proline)',
            'CCG': '프롤린 (Proline)',
            'CAU': '히스티딘 (Histidine)',
            'CAC': '히스티딘 (Histidine)',
            'CAA': '글루타민 (Glutamine)',
            'CAG': '글루타민 (Glutamine)',
            'AAU': '아스파라긴 (Asparagine)',
            'AAC': '아스파라긴 (Asparagine)',
            'AAA': '라이신 (Lysine)',
            'AAG': '라이신 (Lysine)',
            'GAU': '아스파르트산 (Aspartic Acid)',
            'GAC': '아스파르트산 (Aspartic Acid)',
            'GAA': '글루탐산 (Glutamic Acid)',
            'GAG': '글루탐산 (Glutamic Acid)',
            'CGU': '아르기닌 (Arginine)',
            'CGC': '아르기닌 (Arginine)',
            'CGA': '아르기닌 (Arginine)',
            'CGG': '아르기닌 (Arginine)',
            'AGA': '아르기닌 (Arginine)',
            'AGG': '아르기닌 (Arginine)',
            // ... 나머지 아미노산-염기 서열 매핑 추가
        };

        let scene, camera, renderer, cube, controls;

        function init3D() {
            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(75, structureContainer.offsetWidth / structureContainer.offsetHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(structureContainer.offsetWidth, structureContainer.offsetHeight);
            structureContainer.appendChild(renderer.domElement);

            const geometry = new THREE.BoxGeometry(1, 1, 1);
            const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
            cube = new THREE.Mesh(geometry, material);
            scene.add(cube);

            camera.position.z = 5;

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.25;
            controls.enableZoom = true;
            controls.autoRotate = true;
            controls.autoRotateSpeed = 2.0;
        }

        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }

        function appendSequence(nucleotide) {
            currentSequence += nucleotide;
            sequenceDisplay.textContent = currentSequence;
        }

        function checkAminoAcid() {
            if (aminoAcidMap.hasOwnProperty(currentSequence)) {
                resultDisplay.textContent = `조합된 아미노산: ${aminoAcidMap[currentSequence]}`;
                updateAminoAcidStructure(currentSequence); // 3D 구조 업데이트
            } else if (currentSequence.length === 3) {
                resultDisplay.textContent = `해당하는 아미노산을 찾을 수 없습니다.`;
                // 3D 모델을 기본 상태로 되돌리거나, 에러를 시각적으로 표시할 수 있습니다.
            } else if (currentSequence.length > 3) {
                resultDisplay.textContent = `염기 서열이 너무 깁니다. 3개의 염기 서열로 이루어진 코돈만 인식합니다.`;
                // 3D 모델을 기본 상태로 되돌리거나, 에러를 시각적으로 표시할 수 있습니다.
            } else {
                resultDisplay.textContent = `3개의 염기 서열을 조합하여 확인해 주세요.`;
                // 3D 모델을 기본 상태로 되돌리거나, 안내 메시지를 표시할 수 있습니다.
            }
        }

        function resetSequence() {
            currentSequence = '';
            sequenceDisplay.textContent = '';
            resultDisplay.textContent = '';
            // 3D 모델을 초기 상태로 되돌리는 코드를 추가할 수 있습니다.
            if (cube) {
                cube.material.color.set(0x00ff00); // 예시: 큐브 색상을 초기화
            }
        }

        function updateAminoAcidStructure(sequence) {
            // 실제로는 서버에서 해당 염기 서열에 맞는 아미노산의 3D 모델링 데이터를 받아와야 합니다.
            // 여기서는 임시로 간단한 큐브의 색상을 변경하는 것으로 대체합니다.
            let aminoAcidName = '';
            for (const name in aminoAcidMap) {
                if (aminoAcidMap[name] === sequence) { // 염기 서열에 해당하는 아미노산 이름을 찾음
                  aminoAcidName = name;
                  break;
                }
            }

            if (aminoAcidName) {
                console.log(`${sequence}에 해당하는 아미노산: ${aminoAcidName}`);
                cube.material.color.set(Math.random() * 0xffffff); // 임의의 색상 변경
            } else {
                console.log(`${sequence}에 해당하는 아미노산을 찾을 수 없습니다.`);
                cube.material.color.set(0xff0000); // 에러 색상
            }
        }

        init3D();
        animate();
    </script>
</body>
</html>
