<template>
  <div class="container-fluid m-4">
    <h2 class="mb-4">Métier à tisser</h2>
    <div class="row">
      <!-- Colonne 1 : Interface utilisateur -->
      <div class="col-lg-2">

        <hr>
        <button class="btn btn-sm btn-secondary mb-2 d-print-none" @click="printPage">🖨 Imprimer</button>

        <hr class="d-print-none">
        <!-- Fils horizontaux -->
        <div class="mb-4">
          <h5>Fils horizontaux</h5>
          <div v-for="(fil, i) in filsHorizontal" :key="'fh-' + i" class="d-flex align-items-center mb-1">
            <div class="d-flex" style="gap:2px;">
              <div
                v-for="(c, j) in fil.couleurs"
                :key="'fhc-' + i + '-' + j"
                :style="{ backgroundColor: c, width: '18px', height: '18px', border: '1px solid #aaa' }"
              ></div>
            </div>
            <span class="ms-2 text-muted" style="font-size: 0.8em;">{{ fil.epaisseur }} em</span>
            <button class="btn btn-sm btn-outline-primary ms-1" @click="openModal('horizontal', i)">✎</button>
            <button class="btn btn-sm btn-outline-danger ms-1" @click="removeFil('horizontal', i)">🗑</button>
          </div>
          <button class="btn btn-sm btn-outline-primary mt-2" @click="openModal('horizontal')">➕ Ajouter un fil</button>
        </div>

        <hr>

        <!-- Fils verticaux -->
        <div class="mb-4">
          <h5>Fils verticaux</h5>
          <div v-for="(fil, i) in filsVertical" :key="'fv-' + i" class="d-flex align-items-center mb-1">
            <div class="d-flex" style="gap:2px;">
              <div
                v-for="(c, j) in fil.couleurs"
                :key="'fvc-' + i + '-' + j"
                :style="{ backgroundColor: c, width: '18px', height: '18px', border: '1px solid #aaa' }"
              ></div>
            </div>
            <span class="ms-2 text-muted" style="font-size: 0.8em;">{{ fil.epaisseur }} em</span>
            <button class="btn btn-sm btn-outline-primary ms-1" @click="openModal('vertical', i)">✎</button>
            <button class="btn btn-sm btn-outline-danger ms-1" @click="removeFil('vertical', i)">🗑</button>
          </div>
          <button class="btn btn-sm btn-outline-primary mt-2" @click="openModal('vertical')">➕ Ajouter un fil</button>
        </div>

        <hr>

        <!-- Paramètres -->
        <div class="mb-4">
          <h5>Paramètres</h5>
          <label class="form-label">Nombre de colonnes (fils verticaux)</label>
          <input type="number" v-model.number="nbColonnes" class="form-control mb-2" />
          <label class="form-label">Nombre de lignes (fils horizontaux)</label>
          <input type="number" v-model.number="nbLignes" class="form-control" />
          <label class="form-label">Échelle</label>
          <input type="number" v-model.number="echelle" class="form-control" />
          <div class="form-check mb-3">
            <input class="form-check-input" type="checkbox" v-model="afficherClous" id="toggleClous">
            <label class="form-check-label" for="toggleClous">Afficher les clous</label>
          </div>
        </div>
      </div>

      <!-- Colonne 2 : Aperçu principal -->
     <div class="col-lg-4">
  <h5 class="mt-3 d-print-none">Aperçu du tissage</h5>
  <div
    v-if="filsHorizontal.length && filsVertical.length"
    class="tissage-preview border position-relative"
    :style="previewContainerStyle"
    ref="preview"
  >
    <!-- SVG tissage tissé -->
    <svg shape-rendering="crispEdges"
            :width="nbColonnes * distanceClousHorizontal + 'em'"
            :height="nbLignes * distanceClousVertical + 'em'"
            class="position-absolute"
            style="top: 0; left: 0; z-index: 1;"
            :viewBox="`0 0 ${Math.round(nbColonnes * distanceClousHorizontal * 10)} ${Math.round(nbLignes * distanceClousVertical * 10)}`"
            preserveAspectRatio="none"
          >
            <!--
            <defs>
              <filter id="ombre-dessous" x="-50%" y="-50%" width="200%" height="200%">
                <feDropShadow dx="0" dy="1" stdDeviation="1" flood-color="black" flood-opacity="0.3" />
              </filter>
            </defs>-->

            <g>
              
              <path
                v-for="(seg, i) in generateWeavingSegments()"
                :key="'weave-' + i"
                v-bind="seg"
                :filter="seg.filter ? seg.filter : undefined"
                vector-effect="non-scaling-stroke"
              />
          

            </g>
          </svg>

    <!-- Clous -->
    <div v-if="afficherClous ">
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
    </div>
  </div>
</div>


      <!-- Colonne 3 : Répétition -->
   
        <div class="col-lg-6 d-flex flex-column d-print-none">
          <h5 class="mt-3">Répétition {{echelle}}x{{ echelle }}</h5>
          <div
            class="flex-fill border"
            :style="{
              backgroundImage: previewDataUrl ? `url(${previewDataUrl})` : 'none',
              backgroundRepeat: 'repeat',
              backgroundSize: `${nbColonnes * distanceClousHorizontal / echelle}em ${nbLignes * distanceClousVertical / echelle}em`,
              backgroundColor: '#fff',
              minHeight: '0'
            }"
          ></div>
        </div>


    </div>

    <!-- MODAL -->
    <div v-if="showModal" class="modal-backdrop" style="position:fixed; top:0; left:0; right:0; bottom:0; background:#00000088; z-index:10;">
      <div class="modal d-block" tabindex="-1" style="z-index:11;">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">{{ editingIndex !== null ? 'Modifier' : 'Ajouter' }} un fil {{ modalType }}</h5>
              <button type="button" class="btn-close" @click="closeModal()"></button>
            </div>
           <div class="modal-body">
              <label class="form-label">Couleur</label>
              <input
                type="color"
                v-model="modalColor"
                class="form-control form-control-color mb-3"
              />

              <label class="form-label">Épaisseur : {{ modalEpaisseur.toFixed(2) }} em</label>
              <input
                type="range"
                v-model.number="modalEpaisseur"
                min="0.2"
                max="1.2"
                step="0.01"
                class="form-range"
              />
              <!-- APERÇU EN DIRECT -->
              <div
                class="my-3 mx-auto"
                :style="{
                  height: modalEpaisseur + 'em',
                  width: '80%',
                  backgroundColor: modalColor,
                  border: '1px solid #000',
                  borderRadius: '4px'
                }"
              ></div>

            </div>
            

            <div class="modal-footer">
              <button class="btn btn-secondary" @click="closeModal">Annuler</button>
              <button class="btn btn-primary" @click="confirmAddFil">OK</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import html2canvas from 'html2canvas';

export default {
  name: 'TissageView',

  data() {
    return {
      filsHorizontal: [
        { couleurs: ['#ff0000'], epaisseur: 1.0 },
        { couleurs: ['#ffaa00'], epaisseur: 1.0 }
      ],
      filsVertical: [
        { couleurs: ['#1000ff'], epaisseur: 1.0 },
        { couleurs: ['#55aaff'], epaisseur: 1.0 }
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
      clous:{"haut": [0,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,1],
            "droite":[0,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1],
            "bas":   [0,1,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,0],
            "gauche":[0,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0,1,1,1,0]
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
 
   printPage() {
      window.print();
    },

   generateWeavingSegments() {
  const underSegments = [];
  const overSegments = [];
  const cellWidth = this.distanceClousHorizontal * 10;
  const cellHeight = this.distanceClousVertical * 10;

  for (let row = 0; row < this.nbLignes; row++) {
    for (let col = 0; col < this.nbColonnes; col++) {
      const x = Math.round(col * cellWidth);
      const y = Math.round(row * cellHeight);

      const motif = (row + col) % 2;

      // === HORIZONTAL (toujours tracé)
      const filH = this.filsHorizontal[row % this.filsHorizontal.length];
      const colorH = filH.couleurs[0];
      const baseH = filH.epaisseur * 10;
      const factorH = 0.6 + 0.4 * Math.random();
      const epH = baseH * factorH;
      const yMid = y + cellHeight / 2;

      const hSeg = {
        d: `M ${x} ${yMid - epH / 2} H ${x + cellWidth} V ${yMid + epH / 2} H ${x} Z`,
        fill: colorH
      };

      // === VERTICAL (toujours tracé)
      const filV = this.filsVertical[col % this.filsVertical.length];
      const colorV = filV.couleurs[0];
      const baseV = filV.epaisseur * 10;
      const factorV = 0.6 + 0.4 * Math.random();
      const epV = baseV * factorV;
      const xMid = x + cellWidth / 2;

      const vSeg = {
        d: `M ${xMid - epV / 2} ${y} V ${y + cellHeight} H ${xMid + epV / 2} V ${y} Z`,
        fill: colorV
      };

      if (motif === 0) {
        // Horizontal au-dessus
        underSegments.push({ ...vSeg, filter: 'url(#ombre-dessous)' });
        overSegments.push(hSeg);
      } else {
        // Vertical au-dessus
        underSegments.push({ ...hSeg, filter: 'url(#ombre-dessous)' });
        overSegments.push(vSeg);
      }

    }
  }

  return [...underSegments, ...overSegments]; // ordre important !
},


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
    fill: fil.couleurs[0],
    transform: `translate(0, ${(lineIndex + 0.5) * this.distanceClousVertical * 10})` // ✅ décalage
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
    fill: fil.couleurs[0],
    transform: `translate(${(colIndex + 0.5) * this.distanceClousHorizontal * 10}, 0)` // ✅ décalage
  };
},

    
    getHorizontalStyle(lineIndex) {
      const fil = this.filsHorizontal[lineIndex % this.filsHorizontal.length];
      const color = fil.couleurs[lineIndex % fil.couleurs.length];
      const offset = (this.distanceClousVertical - fil.epaisseur) / 2;
      return {
        top: `${lineIndex * this.distanceClousVertical + offset}em`,
        left: 0,
        height: `${fil.epaisseur}em`,
        width: '100%',
        backgroundColor: color,
        zIndex: 1
      };
    },
    getVerticalStyle(colIndex) {
      const fil = this.filsVertical[colIndex % this.filsVertical.length];
      const color = fil.couleurs[colIndex % fil.couleurs.length];
      const offset = (this.distanceClousHorizontal - fil.epaisseur) / 2;
      return {
        left: `${colIndex * this.distanceClousHorizontal + offset}em`,
        top: 0,
        width: `${fil.epaisseur}em`,
        height: '100%',
        backgroundColor: color,
        zIndex: 2
      };
    },
    removeFil(type, index) {
      if (type === 'horizontal') this.filsHorizontal.splice(index, 1);
      else this.filsVertical.splice(index, 1);
      this.updatePreview();
    },
    openModal(type, index = null) {
      this.modalType = type;
      this.editingIndex = index;
      if (index !== null) {
        const fil = type === 'horizontal' ? this.filsHorizontal[index] : this.filsVertical[index];
        this.modalCouleurs = [...fil.couleurs];
        this.modalEpaisseur = fil.epaisseur;
        this.modalColor = fil.couleurs[0];
      } else {
        this.modalCouleurs = [];
        this.modalEpaisseur = 0.6;
        this.modalColor = '#000000';
      }
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
      this.editingIndex = null;
    },
    addColorToModal() {
      if (!this.modalCouleurs.includes(this.modalColor)) {
        this.modalCouleurs.push(this.modalColor);
      }
    },
    confirmAddFil() {
  const fil = {
    couleurs: [this.modalColor],
    epaisseur: this.modalEpaisseur
  };

  if (this.modalType === 'horizontal') {
    if (this.editingIndex !== null) {
      this.filsHorizontal.splice(this.editingIndex, 1, fil);
    } else {
      this.filsHorizontal.push(fil);
    }
  } else {
    if (this.editingIndex !== null) {
      this.filsVertical.splice(this.editingIndex, 1, fil);
    } else {
      this.filsVertical.push(fil);
    }
  }

  this.closeModal();
  this.updatePreview();
},

    updatePreview() {
      this.$nextTick(() => {
        const el = this.$refs.preview;
        if (!el) return;
        html2canvas(el).then(canvas => {
          this.previewDataUrl = canvas.toDataURL();
        });
      });
    }
  },
  mounted() {
    this.updatePreview();

    if (window.matchMedia && window.matchMedia('print').matches) {
      this.distanceClousHorizontal = 0.6;
      this.distanceClousVertical = 0.6;
    }
  },
  watch: {
    filsHorizontal: 'updatePreview',
    filsVertical: 'updatePreview',
    nbColonnes: 'updatePreview',
    nbLignes: 'updatePreview',
    distanceClousHorizontal: 'updatePreview',
    distanceClousVertical: 'updatePreview'
  }
};
</script>

<style scoped>

html, body, .container-fluid, .row {
  height: 100%;
}


.flex-fill {
  flex: 1 1 auto;
  min-height: 0;
}



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

.tissage-preview svg {
  image-rendering: pixelated;
}



@media print {
  body, html {
    margin: 0;
    padding: 0;
  }

  .container-fluid {
    margin: 0 !important;
    padding: 0 !important;
    zoom: 75%; /* Réduit un peu pour tout faire tenir */
  }

  .row {
    display: flex;
    flex-wrap: nowrap;
  }

  .col-lg-2, .col-lg-4, .col-lg-6 {
    flex: 0 0 auto;
    width: 33.33% !important; /* Pour les forcer à être côte à côte */
    max-width: 33.33% !important;
  }

  .d-print-none {
    display: none !important;
  }

  .modal,
  .modal-backdrop {
    display: none !important;
  }

  .tissage-preview {
    page-break-inside: avoid;
  }

  .tissage-preview {
    overflow: visible !important;
  }

   /* forcer l’impression des fonds et bordures */
  *, *::before, *::after {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
  }
  

  @page {
    size: A4 landscape; /* Optionnel : imprime en mode paysage */
    margin: 1cm;
  }
}



</style>
