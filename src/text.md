소개
우리는 이미 콘텐츠를 만들기에 충분한 기본 사항을 알고 있습니다. 첫 번째 프로젝트의 경우 ilithya가 멋진 포트폴리오 https://www.ilithya.rocks/를 사용하여 수행한 작업을 다시 만들고 장면 중앙에 물체가 떠다니는 큰 3D 텍스트가 있습니다.

이 포트폴리오는 Three.js를 배울 때 초기에 무엇을 할 수 있는지 보여주는 훌륭한 예입니다. 간단하고 효율적이며 멋지게 보입니다.

Three.js는 이미 TextGeometry 클래스를 사용하여 3D 텍스트 기하 도형을 지원합니다 . 문제는 글꼴을 지정해야 하며 이 글꼴은 서체라는 특정 json 형식이어야 한다는 것입니다.

라이선스에 대해서는 이야기하지 않겠지만 개인적인 용도가 아닌 한 해당 글꼴을 사용할 수 있는 권한이 있어야 합니다.

서체 글꼴을 얻는 방법
해당 형식의 글꼴을 가져오는 방법에는 여러 가지가 있습니다. 먼저 https://gero3.github.io/facetype.js/ 와 같은 변환기를 사용하여 글꼴을 변환할 수 있습니다 . 파일을 제공하고 변환 버튼을 클릭해야 합니다.

/node_modules/three/examples/fonts/폴더 에 있는 Three.js 예제에서도 글꼴을 찾을 수 있습니다 . 해당 글꼴 /static/을 가져 와서 폴더에 넣거나 json이고 .json파일이 .jsWebpack의 파일 처럼 지원 되기 때문에 JavaScript 파일에서 직접 가져올 수 있습니다 .

import typefaceFont from 'three/examples/fonts/helvetiker_regular.typeface.json'
자바스크립트
우리는 이 두 가지 기술을 혼합하여 열고 /node_modules/three/examples/fonts/, helvetiker_regular.typeface.json및 LICENSE파일을 가져와서 /static/fonts/폴더(생성해야 함)에 넣습니다 .

이제 /fonts/helvetiker_regular.typeface.json기본 URL 끝에 쓰기만 하면 글꼴에 액세스할 수 있습니다 .

글꼴 로드
글꼴을 로드하려면 FontLoader 라는 새 로더 클래스를 사용해야 합니다 . 이 로더는 TextureLoader 처럼 작동합니다 . textureLoader부분 뒤에 다음 코드를 추가합니다 (다른 글꼴을 사용하는 경우 경로를 변경하는 것을 잊지 마십시오).

/**
 * Fonts
 */
const fontLoader = new THREE.FontLoader()

fontLoader.load(
    '/fonts/helvetiker_regular.typeface.json',
    (font) =>
    {
        console.log('loaded')
    }
)
자바스크립트
'loaded'콘솔에 들어가야 합니다 . 그렇지 않은 경우 이전 단계를 확인하고 콘솔에서 잠재적 오류를 검색합니다.

이제 font함수 내부의 변수 를 사용하여 글꼴에 액세스할 수 있습니다. TextureLoader 와 달리 성공 함수 안에 나머지 코드를 작성해야 합니다.

지오메트리 생성
앞서 말했듯이 TextGeometry 를 사용할 것 입니다. 문서 페이지 의 예제 코드에 주의 하십시오 . 값은 우리 장면의 값보다 훨씬 큽니다.

성공 함수 안에 코드를 작성해야 합니다.

fontLoader.load(
    '/fonts/helvetiker_regular.typeface.json',
    (font) =>
    {
        const textGeometry = new THREE.TextGeometry(
            'Hello Three.js',
            {
                font: font,
                size: 0.5,
                height: 0.2,
                curveSegments: 12,
                bevelEnabled: true,
                bevelThickness: 0.03,
                bevelSize: 0.02,
                bevelOffset: 0,
                bevelSegments: 5
            }
        )
        const textMaterial = new THREE.MeshBasicMaterial()
        const text = new THREE.Mesh(textGeometry, textMaterial)
        scene.add(text)
    }
)
자바스크립트
/assets/lessons/13/step-01.png

개선이 필요한 흰색 3D 텍스트가 표시됩니다.

먼저 큐브를 제거하십시오. 그 목적은 모든 것이 제대로 작동하는지 확인하는 것이었습니다.

/assets/lessons/13/step-02.png

멋진 것을 보고 싶다면 wireframe: true자료에 추가 하세요.

const textMaterial = new THREE.MeshBasicMaterial({ wireframe: true })
자바스크립트
/assets/lessons/13/step-03.png

이제 지오메트리가 어떻게 생성되는지 볼 수 있으며 많은 삼각형이 있습니다. 텍스트 지오메트리를 만드는 것은 컴퓨터에서 길고 어렵습니다. 너무 여러 번 수행하는 것을 피하고 curveSegments및 bevelSegments속성 을 줄여 형상을 가능한 한 낮은 폴리로 유지합니다 .

wireframe세부 수준에 만족하면 제거 하십시오.

텍스트 중앙에
텍스트를 가운데에 맞추는 방법에는 여러 가지가 있습니다. 이를 수행하는 한 가지 방법은 경계를 사용하는 것입니다. 경계는 해당 지오메트리가 차지하는 공간을 알려주는 지오메트리와 관련된 정보입니다. 상자나 구일 수 있습니다.

/assets/lessons/13/boundings.png

실제로 이러한 경계를 볼 수는 없지만 Three.js가 객체가 화면에 있는지 쉽게 계산할 수 있도록 도와줍니다. 그렇지 않으면 객체가 렌더링되지도 않습니다. 이것은 절두체 컬링(frustum culling)이라고 하지만 이 강의의 주제는 아닙니다.

우리가 원하는 것은 이 경계를 사용하여 지오메트리의 크기를 알고 중앙에 맞추는 것입니다. 기본적으로 Three.js는 구 경계를 사용합니다. 우리가 원하는 것은 더 정확하게 말하면 상자 경계입니다. 그렇게 하려면 computeBoundingBox()기하학 을 호출하여 이 상자 경계를 계산하도록 Three.js에 요청할 수 있습니다 .

textGeometry.computeBoundingBox()
자바스크립트
boundingBox지오메트리 의 속성으로 이 상자를 선택할 수 있습니다 .

console.log(textGeometry.boundingBox)
자바스크립트
결과는 속성과 속성 이 있는 Box3 이라는 개체 입니다. 속성에 있지 우리가 예상 한 수있다. 이는 and 때문 이지만 지금은 무시할 수 있습니다.minmaxmin0bevelThicknessbevelSize

이제 측정값이 있으므로 개체를 이동할 수 있습니다. 메쉬를 이동하는 대신 전체 형상을 이동합니다. 이렇게 하면 메시는 여전히 장면의 중앙에 있지만 텍스트 형상도 메시 내부의 중앙에 있습니다.

이렇게 하려면 translate(...)메서드 바로 뒤에 있는 지오메트리 에서 메서드를 사용할 수 있습니다 computeBoundingBox().

textGeometry.translate(
    - textGeometry.boundingBox.max.x * 0.5,
    - textGeometry.boundingBox.max.y * 0.5,
    - textGeometry.boundingBox.max.z * 0.5
)
자바스크립트
/assets/lessons/13/step-04.png

텍스트 중심으로해야하지만, 당신은 매우 정확하고 싶다면, 당신은 또한 빼한다 bevelSize이다 0.02:

textGeometry.translate(
    - (textGeometry.boundingBox.max.x - 0.02) * 0.5, // Subtract bevel size
    - (textGeometry.boundingBox.max.y - 0.02) * 0.5, // Subtract bevel size
    - (textGeometry.boundingBox.max.z - 0.03) * 0.5  // Subtract bevel thickness
)
자바스크립트
여기서 수행한 작업 center()은 기하학 에서 메서드를 호출하여 실제로 훨씬 더 빠르게 수행할 수 있습니다 .

textGeometry.center()
GLSL
훨씬 쉽죠? 직접 수행하는 요점은 경계 및 절두체 컬링에 대해 배우는 것이었습니다.

매트캡 소재 추가
이제 텍스트에 멋진 자료를 추가할 차례입니다. 우리는 MeshMatcapMaterial 이 멋지고 성능이 뛰어 나기 때문에 사용할 것 입니다.

먼저 matcap 텍스처를 선택하겠습니다. 우리는 /static/textures/matcaps/폴더 에 있는 matcaps를 사용할 것이지만 여러분 자신의 matcaps를 자유롭게 사용해도 됩니다.

https://github.com/nidorx/matcaps 리포지토리에서 다운로드할 수도 있습니다 . 그것을 선택하는 데 너무 많은 시간을 소비하지 마십시오! 개인적인 용도가 아닌 경우 사용 권한이 있는지 확인하십시오. 고해상도 텍스처가 필요하지 않으며 256x256충분해야 합니다.

이제 코드에 이미 있는 TextureLoader 를 사용하여 텍스처를 로드할 수 있습니다 .

const matcapTexture = textureLoader.load('/textures/matcaps/1.png')
자바스크립트
이제 못생긴 MeshBasicMaterial 을 아름다운 MeshMatcapMaterial로 바꾸고matcapTexture 변수를 matcap속성 과 함께 사용할 수 있습니다.

const textMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })
자바스크립트
/assets/lessons/13/step-05.png

멋진 자료와 함께 멋진 텍스트가 있어야 합니다.

개체 추가
주변에 떠 있는 개체를 추가해 보겠습니다. 그렇게 하기 위해 루프 함수 내부에 하나의 도넛을 만들 것입니다.

성공 함수에서 text부분 바로 뒤에 루프 함수를 추가합니다.

for(let i = 0; i < 100; i++)
{

}
자바스크립트
우리는 이것을 성공 함수 외부에서 수행할 수 있었지만 잠시 후에 보게 될 정당한 이유 때문에 텍스트와 객체가 함께 생성되어야 합니다.

이 루프 에서 텍스트 및 메시 와 동일한 재질인 TorusGeometry (예: 도넛의 기술 이름)를 만듭니다 .

for(let i = 0; i < 100; i++)
{
    const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
    const donutMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })
    const donut = new THREE.Mesh(donutGeometry, donutMaterial)
    scene.add(donut)
}
자바스크립트
/assets/lessons/13/step-06.png

도넛 100개를 한 곳에서 모두 사야 합니다.

위치에 임의성을 추가해 보겠습니다.

donut.position.x = (Math.random() - 0.5) * 10
donut.position.y = (Math.random() - 0.5) * 10
donut.position.z = (Math.random() - 0.5) * 10
자바스크립트
/assets/lessons/13/step-07.png

현장에 100개의 도넛을 분산시켜야 합니다.

회전에 임의성을 추가합니다. 3개의 축을 모두 회전할 필요가 없으며 도넛이 대칭이므로 반 회전이면 충분합니다.

donut.rotation.x = Math.random() * Math.PI
donut.rotation.y = Math.random() * Math.PI
자바스크립트
/assets/lessons/13/step-08.png

도넛은 모든 방향으로 회전해야 합니다.

마지막으로 척도에 임의성을 추가할 수 있습니다. 그러나 조심하십시오. 3개의 축( x, y, z) 모두에 대해 동일한 값을 사용해야 합니다 .

const scale = Math.random()
donut.scale.set(scale, scale, scale)
자바스크립트
/assets/lessons/13/step-09.png

최적화
우리의 코드는 최적화되어 있지 않습니다. 이전 강의에서 보았듯이 여러 Meshes 에서 동일한 재질을 사용할 수 있지만 동일한 지오메트리를 사용할 수도 있습니다.

donutGeometry및 donutMaterial루프 밖으로 이동 :

const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
const donutMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })

for(let i = 0; i < 100; i++)
{
    // ...
}
자바스크립트
동일한 결과를 얻어야 하지만 더 멀리 갈 수 있습니다. 의 재질 text은 와 동일합니다 donut.

를 제거하고 by donutMaterial이름을 변경하고 및 둘 모두에 사용 합니다 .textMaterialmaterialtextdonut

const material = new THREE.MeshMatcapMaterial({ matcap: matcapTexture })

// ...

const text = new THREE.Mesh(textGeometry, material)

// ...

for(let i = 0; i < 100; i++)
{
    const donut = new THREE.Mesh(donutGeometry, material)

    // ...
}
자바스크립트
더 나아갈 수도 있지만 최적화에 대한 전용 강의가 있습니다.

더 나아가
원하는 경우 더 많은 모양을 추가하고 애니메이션을 적용하고 다른 매트캡을 시도할 수도 있습니다.