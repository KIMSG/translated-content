---
title: WebXR 設定の照明
slug: Web/API/WebXR_Device_API/Lighting
tags:
  - 3D
  - API
  - Graphics
  - Light
  - Shading
  - Shadows
  - WebGL
  - WebXR
  - WebXR Device API
  - lighting
  - rendering
translation_of: Web/API/WebXR_Device_API/Lighting
---
{{DefaultAPISidebar("WebXR Device API")}}

[WebXR Device API](/ja/docs/Web/API/WebXR_Device_API) は、シーンのすべてのレンダリング、テクスチャリング、およびライティング（照明）を実行するために他のテクノロジー（つまり、[WebGL](/ja/docs/Web/API/WebGL_API) とそれに基づくフレームワーク）に依存しているため、WebXR 設定またはシーンには、他の WebGL で生成されたディスプレイと同じ一般的な照明の概念が適用されます。

ただし、特に拡張現実（AR）アプリケーションの場合、照明のコードを作成する際に留意すべき問題と詳細があります。 このガイドでは、これらのトピックについて説明します。 そして、この記事では、照明が一般的にどのように機能するかについて簡単に説明しますが、照明のチュートリアルや、適切に照明された 3D シーンを作成する方法のガイドではありません。

## フラッシュバック: 3D での照明のシミュレーション

この記事は 3D シーンを照明するための包括的なガイドではありませんが、照明が一般的にどのように機能するかについて簡単に思い出させるのに役立ちます。 基本的に、仮想シーンでの照明のシミュレーションには、シーン内の各オブジェクトと相互作用して反射した後、各光源からの光が目で受け取る量を計算することが含まれます。

### 光の反射

**反射角が入射角にどのように対応するかを示す図。**
![反射角が入射角にどのように対応するかを示す図。](law-of-reflection.svg)

私たちが見るすべてのオブジェクトは、オブジェクトが光を放出または反射する（あるいはその両方）ために見えます。 入ってくる光線（**入射光線**、incident ray）は、**入射角**（angle of incidence）と呼ばれる角度で到達します。 入射角 Θi は、入射光線と表面の{{interwiki("wikipedia", "法線ベクトル")}}の間の角度です。

粗い表面の場合、光はあらゆる方向に均等に反射されます。 ただし、光沢のある鏡のような表面は、法線ベクトルの反対側にあることを除いて、**反射角**（angle of reflection） Θr が入射角に等しい方向にほとんどの光を反射します。 **反射光線**（reflected ray）は、法線からその角度で出発します。 これが**{{interwiki("wikipedia", "鏡面反射", "反射の法則")}}**（law of reflection）です。 これは、シーンのシェーディングに関係する多くの基盤であり、さまざまな種類の光源の振る舞いの観点で役に立ちます。

もちろん、反射光の光線の色は、光が表面と相互作用するために強度や色相が変化する可能性がありますが、角度は常に同じです。

### 光源の構成要素

光源には 3 つの主要な構成要素があります。 各構成要素は本質的に一種の光です。

ビューアーの画面またはヘッドセットに表示されるオブジェクトとそのピクセルの色と明るさに影響を与える可能性のある 3 種類の光があります。

#### 環境光

**環境光**（ambient light）は、定義された光源からではなく、シーン全体に存在する光です。 この光は、シーン内のあらゆる表面にあらゆる方向から同じ強度で到達し、あらゆる方向に均等に反射します。 その結果、環境光によって適用される効果は、シーン全体で普遍的に等しくなります。

**環境照明のみの球。 球の奥行きを示すための陰影がまったくないことに注意してください。**
![環境照明のみを持つ球。 球の奥行きを示すための陰影がまったくないことに注意してください。](sphere-ambient-light-only.jpg)

環境光の効果は、光源の強度にピクセルの位置での表面の反射率を単純に乗算することによって計算されます。 シーン内のあらゆるピクセルの色と強度は、シーン内のどこにあるか、または向きに関係なく、まったく同じように影響を受けます。 環境光は通常、シーン全体に影響を与えますが、影の部分が暗くなりすぎるのを防ぐために存在します。 ただし、シーン内の環境光の量は非常に少なくする必要があります。

光の跳ね返りと散乱はリアルタイムで計算するのにコストがかかる可能性があるため、特に複数の光源が関係している場合は、光散乱の真の効果を実際に計算するのではなく、環境照明を使用してシーン内の他のすべての光源によって引き起こされる散乱光をシミュレートするのが一般的です。 ただし、これを行う場合は、環境光をシーンの照明の実際の効果に一致させるように注意する必要があります。

環境光を使用して、シーンに色合いを適用することもできます。 例えば、プレーヤーが黄色がかった特別な眼鏡を持っているゲームでは、黄色の環境光を追加できます。

#### 拡散光

**拡散光**（diffuse light）は、表面から均一かつ指向性を持って放射または反射する光です。 これは私たちが通常目にする光の大部分です。 拡散光は特定の位置または方向から来て、影を落とします。 その指向性により、拡散光に面しているオブジェクトの面は、他の面よりも明るくなります。

**土星で 5 番目に大きい月テティスは、左下から日光を浴びています。**
![土星で5番目に大きい月テティスは主に太陽に照らされており、土星からの光が反射しています。 これは拡散照明です。](tethys.jpg)

拡散光の強度は{{interwiki("wikipedia", "入射角")}}（angle of incidence、光が表面に到達する方向を表すベクトルと表面の法線ベクトルまたは表面に垂直なベクトルとの間の角度）に依存するため、オブジェクトによって反射する光の強度または明るさは、光源に対する表面の向きによって異なります。

#### 鏡面反射光

鏡面反射光（specular light）は、宝石、目、光沢のあるカップや皿などの反射オブジェクトのハイライトを構成する光です。 鏡面反射光は、光源が表面に最も直接当たる点の表面に明るいスポットまたは正方形として現れる傾向があります。

**NASA のカッシーニ宇宙船が撮影した写真。 土星の月タイタンの表面にある液体メタンの湖からの光の鏡面反射を示しています。**
![NASA のカッシーニ宇宙船が撮影した写真。 土星の月タイタンの表面にある液体メタンの湖からの光の鏡面反射を示しています。](specularlight-titan.jpg)

すべての光源は、環境光、拡散光、および/または鏡面反射光の組み合わせによって表されます。 WebGL シェーダープログラムは、各光源の色、指向性、明るさ、およびその他の要素を取得し、各ピクセルの最終的な色を計算します。

### 光源の種類

光源には 4 つの基本的な種類があります。 それぞれに、描画されるオブジェクトからの距離と光波の指向性によって光源が特定の特性を帯びる仮想光源が含まれます。 ほとんどの場合、これらの光源タイプの 1 つまたは複数を使用して、現実世界の光源をシミュレートできます。

#### 環境光源

**環境光源**（ambient light source）は、シーン内の環境光のレベルと色を表す光源です。 シーンにはこれらが複数存在する場合がありますが、それぞれが常にすべてのピクセルに均等に影響するため、これらを 1 つにまとめることで、パフォーマンスをわずかに向上させることができます。

環境光源は通常、シーン内のどのオブジェクトにも対応しておらず、現実世界の類似物もありません。

#### 指向性光源

**指向性光源**（directional light source、平行光源）は、特定の方向から来る光源ですが、特定の光源からは来ないため、放出される光線は互いに平行です。 また、光の強さは距離によって変化しません。 これは、指向性光によって投影される影が非常にシャープであり、光と影の間で本質的に瞬時に遷移することを意味します。

**地球と月は両方とも太陽の指向性照明によって半分照らされています。**
![ガリレオ宇宙船が約630万キロメートル離れた場所から撮影した写真で、地球と月の両方が太陽に半分照らされています。](earthandmoon.jpg)

指向性光の最も一般的な例は太陽です。 太陽は実際には単一の（大きな）オブジェクトですが、非常に遠くにあるため、太陽からの光線は基本的に平行です。 太陽光は実際には距離とともに強度が低下しますが、変化率は非常に低く、広大な距離でのみ認識されるため、太陽光の強度変化率は通常、3D シーンのレンダリングには関係ありません。

#### 点光源

**点光源**（point light source）は、特定の場所に配置され、すべての方向に均等に外側に放射する光源です。 電球、ろうそくなどは点光源の例です。 オブジェクトが点光源に近いほど、そのオブジェクトに照射される光は明るくなります。 ポイントライト（point light）の明るさが低下するレートは**減衰**（attenuation）と呼ばれ、WebGL やその他の照明システムの光源の構成可能な機能です。

反射の法則と光線の明るさが距離とともに減少するという事実との間で、点光源から放出されて反射する光は、光源に最も近い点で最も明るくなり、光源から離れるほど暗くなる傾向があります。 表面が平らであっても、光源に最も近い点が中心で、法線から離れるように角度が変化するにつれて光線はますます長くなります。

#### スポット光源

**スポット光源**（spot light source）または**スポットライト**（spotlight）は、特定の位置に配置され、その方向ベクトルの方向に光の円錐を放出する光源です。 テーパリングレートパラメーターは、光の円錐の端で光の明るさがどれだけ速く落ちるかを定義し、ポイントライトと同様に、減衰パラメーターは、光が距離とともにどのようにフェードするかを制御します。

**夜に漆喰の壁を照らすスポットライトの写真。**
![夜に漆喰の壁を照らすスポットライトの写真。](spotlight-on-stucco.jpg)

光の円錐の端では、光は表面にまったく影響を与えなくなります。

#### 照明の計算コスト

シーンを目に見えるようにするには、何らかの照明が含まれている必要があるため、すべてまたはほぼすべてのシーンに少なくとも 1 つの光源があり、非常に多くの光源を持っているかもしれません。 各光源は、表示される各ピクセルの最終的な色と明るさを決定するために必要な計算量を大幅に増やします。 これらの光源の種類のそれぞれに対してシェーディングを実行することは、その前のものよりも計算量が多くなります。 したがって、環境光を適用するのが最も費用がかからず、次に指向性光源、ポイントライト、最後にスポットライトが続きます。

さらに、照明がより正確になるように設計されているほど、計算コストが高くなります。 陰影詳細の増加、体積光（つまり、太陽光のビームや空のスポットライトのビームなど、空中で見ることができる照明）、およびその他の照明効果は、シーンにリアリズムと美しさを追加できますが、シーンが GPU を圧倒しないように注意が必要です。

### 照らされたピクセルの色の計算

一部のグラフィックライブラリーには光源オブジェクトのサポートが含まれており、照明効果を自動的に計算して適用しますが、WebGL には含まれていません。 幸い、照明を独自の頂点シェーダーとフラグメントシェーダーで適用することはそれほど難しくありません。

シーン内のポリゴンごとに、**頂点シェーダー**（vertex shader）のプログラムが頂点の色を決定し、**フラグメントシェーダー**（fragment shader）は、割り当てられたテクスチャー、任意の色合いまたは効果、およびその他の視覚データから適切なテクセルを組み合わせて、ポリゴン内の各ピクセルを生成します。 ピクセルがフレームバッファーに格納される前に、シーンの照明が考慮され、ピクセルに適切に適用されるのはこのときです。

最終的なレンダリングされたシーンの各ピクセルの色は、次のような要素を考慮した複雑な計算を使用して計算されます。

- 画面のピクセルに対応する**テクスチャー要素**（texture element、オブジェクトにマップされたテクスチャー内のピクセル。 **テクセル**（texel）とも呼ばれます）の色。 オブジェクトのジオメトリー、各ポリゴンに対するビューアーの位置と方向などが与えられます。
- ビューアーの位置と距離。
- 表面の材質（マテリアル）と反射率。
- ターゲット位置での表面の凹面または凸面。
- シーン内の各光源の位置、色、指向性、および明るさ。
- シーン内の環境光の色と明るさ。 これは、シーン全体に均等に適用される光であり、光源がないため、影や明るさの変化がありません。
- シーン内の他の表面で反射した光の効果。 反射光の色、指向性、明るさは、光が触れるピクセルの色に影響します。

WebGL で照明を実行する方法の詳細については、[WebGL でのライティング](/ja/docs/Web/API/WebGL_API/Tutorial/Lighting_in_WebGL)の記事を参照してください。

## 複合現実コンテンツの照明の問題

シーンの照明中に対処する必要のある通常の問題に加えて、VR または AR のユースケースでは、シェーダーを作成する際の懸念事項が追加されます。 このセクションでは、シーンを構築、レンダリング、および照明するときに考慮すべきいくつかの基本的な複合現実の照明のガイドラインを提供します。 これらのいくつかは他の 3D 設定でも役立ちますが、ほとんどは仮想現実に固有であり、場合によっては拡張現実に固有です。

シーンは、人物またはそのアバターが存在する可能性のある設定を表すことを目的としているため、光源の配置と提示に関して、ある程度の一貫性とリアリズムを実現するように努める必要があります。 明らかに、このガイドラインには例外があります。 例えば、シーンが異世界または異星人の設定を表している場合や、不安な視覚効果を作成することが目標である場合などです。

### 光源配置のリアリズム

可能であれば、仮想光源を実際に存在するオブジェクトに対応させるようにしてください。 頭上の照明が必要な仮想の部屋がある場合は、光源の場所に天井ランプのモデルを用意します。 例外として、設定にベースライン量の照明を追加するだけの環境照明や、指向性光である太陽（つまり、すべての光線が平行で、空のどこかから来てシーン内のどこかで終わる光源）などがあります。

作成しようとしている設定とムードに合わせて、現実的な場所に光源を配置してください。 自然に照らされた現実世界の設定のように感じることを目的としたシーンには、スタジオ照明がありません。 太陽光や、シーン内のオブジェクトや水から反射した光などがありますが、シーン内のオブジェクトや人の顔に向けられたランプはありません。

### 光とプレイヤーの相互作用のリアリズム

光源がシーン内にある場合は、ビューアーのアバターが光源と物理的に交差しないようにする必要があります。 結果は…奇妙なものになるかもしれません。

ビューアーのアバターが物理的な形をとることを意図している場合、ビューアーがそれを決して見ることができない場合でも、光がアバターと正しく相互作用するように、モデルを持っている必要があります。 最低限でも、これはアバターが適切な影を落とす必要があることを意味しますが、アバターが見えるかどうかや、モデルのマテリアル、テクスチャー、その他の属性（特に反射率を含む）などの要因によっては、アバターも光を反射する必要があり、反射した光の着色に影響を与える可能性があります。

### 拡張現実におけるリアリズム

拡張現実は、仮想オブジェクトが独自の光源を持つ物理的な世界内に存在する必要があるため、オブジェクトを照明するのがさらに複雑になります。 そのため、照明を現実世界の光源にできるだけ一致させるようにしてください。 これは、[照明推定](#lighting_estimation)と呼ばれる手法を使用して行われます。

逆に、それ自体が光源である仮想オブジェクトは、現実世界の設定にその照明を当てることができるコードを作成する準備ができていない限り、使用しないようにする必要があります。 現実世界のオブジェクトに光を当てるのは、基本的に影を付けるのとは逆です。 それは可能ですが、それほど広く実装されていません。

## 照明推定

**照明推定**（lighting estimation）は、拡張現実プラットフォームで使用される手法であり、シーン内の仮想オブジェクトの照明を、ビューアーを取り巻く現実世界の照明に一致させようとします。 これには、さまざまなセンサー（加速度計とコンパスがある場合はそれを含む）、カメラ、および場合によっては他のセンサーから取得される可能性のあるデータの収集が含まれます。 その他のデータは [Geolocation API](/ja/docs/Web/API/Geolocation_API) を使用して収集され、このすべてのデータはアルゴリズムと機械学習エンジンに送られ、推定された照明情報が生成されます。

現在、WebXR は照明推定のサポートを提供していません。 ただし、[仕様は現在 W3C の支援の下で起草](https://github.com/immersive-web/lighting-estimation)されています。 仕様の GitHub リポジトリに含まれている[説明文書](https://github.com/immersive-web/lighting-estimation/blob/master/lighting-estimation-explainer.md)で、提案された API のすべてと、照明推定の概念についてかなりの量を学ぶことができます。

本質的に、照明推定は、光源と、シーン内のオブジェクトの形状と方向、およびそれらを構成している材質に関する情報を収集し、現実世界の照明とほぼ一致する仮想光源オブジェクトの作成に使用できるデータを返します。

特に提案された API のコンテキストで、照明推定がどのように機能するかの詳細は、現時点では範囲外です。 API が安定したら、このドキュメントを詳細で更新します。

## セキュリティとプライバシーの懸念

現実世界のデータを使用して仮想オブジェクトに照明を生成および適用するために、このすべてのデータを収集することに関連する潜在的なセキュリティ問題がいくつかあります。

もちろん、多くの AR アプリケーションは、ユーザーがどこにいるかをかなり明確にします。 ユーザーが*ルーブルのツアー*というアプリを実行している場合、ユーザーがフランスのパリにあるルーブル美術館（[Musée du Louvre](https://louvre.fr/)）にいる可能性が非常に高くなります。 ただし、ブラウザーは、ユーザーの同意なしにユーザーの居場所を物理的に特定することを困難にするために、いくつかの手順を実行する必要があります。

### Ambient Light Sensor API

[Ambient Light Sensor API](/ja/docs/Web/API/Ambient_Light_Sensor_API) を使用して光データを収集すると、さまざまな潜在的なプライバシー問題が発生します。

- 照明情報は、ユーザーの周囲やデバイスの使用パターンに関する情報をウェブに漏洩する可能性があります。 このような情報は、ユーザープロファイリングおよび行動分析データを強化するために使用できます。
- 2 つ以上のデバイスが同じサードパーティのスクリプトを使用するコンテンツにアクセスする場合、そのスクリプトを使用して、照明情報と、それが時間の経過とともにどのように変化するかを相互に関連付けて、デバイス間の空間的関係を決定しようとすることができます。 これは、理論的には、例えば、デバイスが大体同じ領域にあることを示すことができます。

### ブラウザーがこれらの問題を軽減する方法

これらのリスクを軽減するために、WebXR Lighting Estimation API 仕様では、ブラウザーは、真の値から多少ずれた照明情報を報告する必要があります。 これを行うには多くの方法があります。

#### 球面調和関数の精度

ブラウザーは、{{interwiki("wikipedia", "球面調和関数")}}（spherical harmonics）の精度を下げることにより、フィンガープリント（fingerprinting）のリスクを軽減できます。 仮想現実または拡張現実のアプリケーションの場合のように、リアルタイムレンダリングを実行する場合、球面調和関数照明（[spherical harmonic lighting](https://en.wikipedia.org/wiki/spherical%20harmonic%20lighting)）を使用して、非常にリアルな影とシェーディングを生成するプロセスを簡素化および加速します。 これらの機能の精度を変更することにより、ブラウザーはデータの一貫性を低下させ、重要なことに、同じ設定であっても 2 台のコンピューターによって生成されるデータを異なるものにします。

#### 方向の照明からの切り離し

測位（geolocation）を使用して方向と潜在的な位置情報を決定する AR アプリケーションでは、その情報が照明の状態に直接相関しないようにすることは、ブラウザーがフィンガープリント攻撃からユーザーを保護できるもう 1 つの方法です。 ユーザーの位置に近い（または近いと主張する）すべてのデバイスでコンパスの方向と光の指向性が同じではないことを確認するだけで、周囲の照明の状態に基づいてユーザーを見つける能力が取り除かれます。

ブラウザーが非常に明るい指向性光源に関する詳細を提供する場合、その光源はおそらく太陽を表しています。 この明るい光源の指向性を時刻と組み合わせて使用 ​​ すると、Geolocation API を使用せずにユーザーの地理的位置を特定できます。 AR シーンの座標がコンパス座標と一致しないようにし、太陽の光の角度の精度を下げることにより、この手法を使用して位置を正確に推定できなくなります。

#### 時間的空間的フィルタリング

建物の自動照明システムを使用して、既知のパターンで光をすばやくオン/オフする攻撃について考えてみます。 適切な予防策がなければ、照明推定データを使用してこのパターンを検出し、ユーザーが特定の場所にいると判断する可能性があります。 これはリモートで実行することも、同じ部屋にいるが他の人も同じ部屋にいるかどうかを判断したい攻撃者が実行することもできます。

照明推定を使用してユーザーに関する情報を許可なく取得できる別のシナリオとしては、光センサーがユーザーのディスプレイに十分近く、ディスプレイの内容によって引き起こされる照明の変化を検出する場合、アルゴリズムを使用して、ユーザーが特定の動画を視聴しているかどうかを判断したり、ユーザーが見ている動画の数を特定する可能性さえあります。

Lighting Estimation API 仕様では、すべての{{Glossary("user agent", "ユーザーエージェント")}}が時間的空間的フィルタリング（temporal and spatial filtering）を実行して、ユーザーの位置を特定したり{{interwiki("wikipedia", "サイドチャネル攻撃")}}（side-channel attacks）を実行したりする目的での有用性を低下させる方法でデータをぼやけさせることを義務付けています。

## 関連情報

- [WebXR Lighting Estimation API の説明](https://github.com/immersive-web/lighting-estimation/blob/master/lighting-estimation-explainer.md)（英語）
- [WebXR Lighting Estimation API Level 1 仕様](https://github.com/immersive-web/lighting-estimation)（英語）
- [シェーダーを用いた WebGL での色の指定](/ja/docs/Web/API/WebGL_API/Tutorial/Using_shaders_to_apply_color_in_WebGL)
- [WebGL でのテクスチャの使用](/ja/docs/Web/API/WebGL_API/Tutorial/Using_textures_in_WebGL)
- [WebGL でのライティング](/ja/docs/Web/API/WebGL_API/Tutorial/Lighting_in_WebGL)
- [GLSL シェーダー](/ja/docs/Games/Techniques/3D_on_the_web/GLSL_Shaders)
