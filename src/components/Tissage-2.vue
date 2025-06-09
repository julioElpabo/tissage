<template>
  <div class="container-fluid m-4">
    <h2 class="mb-4">Métier à tisser</h2>
    <div class="row">
      <!-- Colonne 1 : Interface utilisateur -->
      <!-- inchangée -->

      <!-- Colonne 2 : Aperçu principal -->
      <div class="col-lg-5">
        <h5 class="mt-3">Aperçu du tissage</h5>
        <div
          v-if="filsHorizontal.length && filsVertical.length"
          class="tissage-preview border position-relative"
          :style="previewContainerStyle"
          ref="preview"
        >
          <!-- SVG tissage -->
          <svg
            :width="nbColonnes * distanceClousHorizontal + 'em'"
            :height="nbLignes * distanceClousVertical + 'em'"
            class="position-absolute"
            style="top: 0; left: 0; z-index: 1;"
            :viewBox="`0 0 ${nbColonnes * distanceClousHorizontal * 10} ${nbLignes * distanceClousVertical * 10}`"
            preserveAspectRatio="none"
          >
            <!-- Fils horizontaux -->
            <g v-for="lineIndex in nbLignes" :key="'svg-horiz-' + lineIndex">
              <path
                :d="generateHorizontalPath(lineIndex - 1).d"
                :fill="generateHorizontalPath(lineIndex - 1).fill"
                :transform="`translate(0, ${(lineIndex - 1) * distanceClousVertical * 10})`"
              />
            </g>

            <!-- Fils verticaux -->
            <g v-for="colIndex in nbColonnes" :key="'svg-vert-' + colIndex">
              <path
                :d="generateVerticalPath(colIndex - 1).d"
                :fill="generateVerticalPath(colIndex - 1).fill"
                :transform="`translate(${(colIndex - 1) * distanceClousHorizontal * 10}, 0)`"
              />
            </g>
          </svg>

          <!-- Clous -->
          <template v-if="afficherClous">
            <div
              v-for="c in clousHautActifs"
              :key="'clou-top-' + c.index"
              class="position-absolute clou"
              :style="{
                top: `-${clouOffset}px`,
                left: `${c.index * distanceClousHorizontal}em`,
                transform: 'translateX(-50%)'
              }"
            ></div>
            <div
              v-for="c in clousBasActifs"
              :key="'clou-bottom-' + c.index"
              class="position-absolute clou"
              :style="{
                top: `${nbLignes * distanceClousVertical}em`,
                left: `${c.index * distanceClousHorizontal}em`,
                transform: 'translateX(-50%)'
              }"
            ></div>
            <div
              v-for="c in clousGaucheActifs"
              :key="'clou-left-' + c.index"
              class="position-absolute clou"
              :style="{
                left: `-${clouOffset}px`,
                top: `${c.index * distanceClousVertical}em`,
                transform: 'translateY(-50%)'
              }"
            ></div>
            <div
              v-for="c in clousDroiteActifs"
              :key="'clou-right-' + c.index"
              class="position-absolute clou"
              :style="{
                left: `${nbColonnes * distanceClousHorizontal}em`,
                top: `${c.index * distanceClousVertical}em`,
                transform: 'translateY(-50%)'
              }"
            ></div>
          </template>
        </div>
      </div>

      <!-- Colonne 3 : Répétition -->
      <div class="col-lg-5">
        <h5 class="mt-3">Répétition {{echelle}}x{{ echelle }}</h5>
        <div
          class="position-relative border"
          :style="{
            width: `${nbColonnes * distanceClousHorizontal * 3 / 3}em`,
            height: `${nbLignes * distanceClousVertical * 3 / 3}em`,
            backgroundImage: `url(${previewDataUrl})`,
            backgroundRepeat: 'repeat',
            backgroundSize: `${nbColonnes * distanceClousHorizontal / echelle}em ${nbLignes * distanceClousVertical / echelle}em`,
            backgroundColor: '#fff'
          }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TissageView',
  data() {
    return {
      filsHorizontal: [
        { couleurs: ['#ff0000'], epaisseur: 0.6 },
        { couleurs: ['#ffaa00'], epaisseur: 0.8 }
      ],
      filsVertical: [
        { couleurs: ['#0000ff', '#00ff00'], epaisseur: 0.6 },
        { couleurs: ['#55aaff'], epaisseur: 0.3 }
      ],
      distanceClousHorizontal: 1.0,
      distanceClousVertical: 1.0,
      nbColonnes: 31,
      nbLignes: 35,
      echelle: 3.0,
      clouOffset: 10,
      showModal: false,
      modalType: 'horizontal',
      modalCouleurs: [],
      modalColor: '#000000',
      modalEpaisseur: 0.6,
      editingIndex: null,
      previewDataUrl: '',
      afficherClous: true,
      clous: {
        haut: [0,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,1],
        droite: [0,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1],
        bas: [0,1,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,0],
        gauche: [0,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0]
      }
    };
  },
  computed: {
    previewContainerStyle() {
      return {
        width: `${this.nbColonnes * this.distanceClousHorizontal}em`,
        height: `${this.nbLignes * this.distanceClousVertical}em`,
        backgroundColor: '#f0f0f0',
        padding: '10px',
        position: 'relative'
      };
    },
    clousHautActifs() {
      return this.clous.haut.map((val, index) => ({ val, index })).filter(c => c.val === 1);
    },
    clousBasActifs() {
      return this.clous.bas.map((val, index) => ({ val, index })).filter(c => c.val === 1);
    },
    clousGaucheActifs() {
      return this.clous.gauche.map((val, index) => ({ val, index })).filter(c => c.val === 1);
    },
    clousDroiteActifs() {
      return this.clous.droite.map((val, index) => ({ val, index })).filter(c => c.val === 1);
    }
  },
  methods: {
    generateHorizontalPath(lineIndex) {
      const baseWidth = this.nbColonnes * this.distanceClousHorizontal * 10;
      const resolution = 50;
      const fil = this.filsHorizontal[lineIndex % this.filsHorizontal.length];
      const base = fil.epaisseur * 10;
      let path = '';
      let topPts = [];
      let botPts = [];

      for (let i = 0; i <= resolution; i++) {
        const x = (i / resolution) * baseWidth;
        const factor = 0.6 + 0.4 * Math.random();
        const ep = base * factor;
        topPts.push(`${x},${-ep / 2}`);
        botPts.unshift(`${x},${ep / 2}`);
      }
      path += `M ${topPts[0]} L ${topPts.slice(1).join(' ')} L ${botPts.join(' ')} Z`;
      return {
        d: path,
        fill: fil.couleurs[0]
      };
    },
    generateVerticalPath(colIndex) {
      const baseHeight = this.nbLignes * this.distanceClousVertical * 10;
      const resolution = 50;
      const fil = this.filsVertical[colIndex % this.filsVertical.length];
      const base = fil.epaisseur * 10;
      let path = '';
      let leftPts = [];
      let rightPts = [];

      for (let i = 0; i <= resolution; i++) {
        const y = (i / resolution) * baseHeight;
        const factor = 0.6 + 0.4 * Math.random();
        const ep = base * factor;
        leftPts.push(`${-ep / 2},${y}`);
        rightPts.unshift(`${ep / 2},${y}`);
      }
      path += `M ${leftPts[0]} L ${leftPts.slice(1).join(' ')} L ${rightPts.join(' ')} Z`;
      return {
        d: path,
        fill: fil.couleurs[0]
      };
    }
  }
};
</script>

<style scoped>
.tissage-preview {
  margin-top: 1rem;
}

.clou {
  width: 10px;
  height: 10px;
  background-color: black;
  border-radius: 50%;
  z-index: 2;
  border: 1px solid white;
}
</style>
