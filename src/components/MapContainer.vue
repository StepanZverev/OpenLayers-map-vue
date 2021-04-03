<template>
  <div>
    <!-- Карта -->
    <div ref="map-root" style="width: 100%; height: 600px"></div>
    <!-- /Карта -->
    <!-- Попап -->
    <div ref="popup" class="ol-popup">
      <a href="#" @click="closePopup" class="ol-popup-closer"></a>
      <div id="popup-content" class="ol-popup-content">
        <v-menu offset-y>
          <template v-slot:activator="{ on }">
            <v-btn
              :ripple="false"
              height="40"
              elevation="0"
              :color="currentPointColor"
              v-on="on"
            >
            </v-btn>
          </template>
          <v-color-picker v-model="currentPointColor"></v-color-picker>
        </v-menu>
        <v-select dense v-model="currentPointType" outlined :items="list">
          <template slot="selection" slot-scope="data">
            <v-icon :color="currentPointColor">{{ "mdi-" + data.item }}</v-icon>
          </template>
          <template slot="item" slot-scope="data">
            <v-icon :color="currentPointColor">{{ "mdi-" + data.item }}</v-icon>
          </template>
        </v-select>
        <v-btn height="40" elevation="0" v-if="isPopupEditMode" @click="editPoint"
          >Edit</v-btn
        >
        <v-btn height="40" elevation="0" v-else @click="addPoint">Add</v-btn>
      </div>
    </div>
    <!-- /Попап -->
  </div>
</template>

<script>
import View from "ol/View";
import Map from "ol/Map";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import VectorLayer from "ol/layer/Vector";
import VectorSource from "ol/source/Vector";
import Feature from "ol/Feature";
import Point from "ol/geom/Point";
import { Style, Icon } from "ol/style";
import Overlay from "ol/Overlay";

import "ol/ol.css";

// Получить стиль метки по её типу
const getFeatureStyle = (type, color) => {
  switch (type) {
    case "circle":
      return new Style({
        image: new Icon({
          color: color,
          src: "circle.png",
        }),
      });
    case "square":
      return new Style({
        image: new Icon({
          color: color,
          src: "square.png",
        }),
      });
    case "triangle":
      return new Style({
        image: new Icon({
          color: color,
          src: "triangle.png",
        }),
      });
  }
};

export default {
  name: "MapContainer",
  components: {},
  data: () => ({
    // Данные для попапа
    list: ["circle", "square", "triangle"],
    currentPointType: "circle",
    currentPointPosition: null,
    currentPointId: null,
    currentPointColor: "#000000",
    isPopupEditMode: false,
    // Данные для карты
    olMap: null,
    vectorLayer: null,
    pointsCoordinates: [],
    overlay: null,
  }),
  mounted() {
    // Создаем попап
    this.overlay = new Overlay({
      element: this.$refs["popup"],
      autoPan: true,
      autoPanAnimation: {
        duration: 250,
      },
    });

    // создаем слой
    this.vectorLayer = new VectorLayer({
      source: new VectorSource({
        features: [],
      }),
    });

    // создаем карту
    this.olMap = new Map({
      target: this.$refs["map-root"],
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
        this.vectorLayer,
      ],
      view: new View({
        zoom: 0,
        center: [0, 0],
        constrainResolution: true,
      }),
      overlays: [this.overlay],
    });

    this.olMap.on("click", (event) => {
      // координаты текущей точки
      const newPointCoordinates = this.olMap.getCoordinateFromPixel(
        event.pixel
      );
      // получаем маркер по координатам клика
      const selectedFeature = this.olMap.getFeaturesAtPixel(event.pixel)[0];

      // если попали в маркер
      if (selectedFeature) {
        // включаем режим редактирования маркера
        this.isPopupEditMode = true;
        // считываем параметры маркера
        const pointProperties = selectedFeature.getProperties();
        // устанавливаем в стейт
        this.currentPointId = pointProperties.id;
        this.currentPointType = pointProperties.type;
        this.currentPointColor = pointProperties.color;
        this.currentPointPosition = selectedFeature
          .getGeometry()
          .getCoordinates();
      } else {
        // иначе: выключаем режим редактироания, записываем координаты в стейт
        this.isPopupEditMode = false;
        this.currentPointPosition = newPointCoordinates;
      }
      // открываем попап
      this.overlay.setPosition(newPointCoordinates);
    });
  },
  methods: {
    // обновить данные карты
    updateSource() {
      const source = this.vectorLayer.getSource();
      const features = [];

      // создаем точки по координатам из стейта
      this.pointsCoordinates.forEach((point, index) => {
        const feature = new Feature({
          geometry: new Point(point.position),
        });
        feature.setStyle(getFeatureStyle(point.type, point.color));
        feature.setProperties({
          id: index,
          type: point.type,
          color: point.color,
        });
        features.push(feature);
      });

      // обновляем
      source.clear();
      source.addFeatures(features);
    },
    // добавить марер
    addPoint() {
      const newPont = {
        type: this.currentPointType,
        color: this.currentPointColor,
        position: this.currentPointPosition,
      };
      this.pointsCoordinates = [...this.pointsCoordinates, newPont];
      // скрываем попап
      this.overlay.setPosition(undefined);
      this.currentPointPosition = null;
      this.updateSource();
    },
    // изменить маркер
    editPoint() {
      const editedPoint = {
        type: this.currentPointType,
        color: this.currentPointColor,
        position: this.currentPointPosition,
      };
      this.pointsCoordinates[this.currentPointId] = editedPoint;
      this.overlay.setPosition(undefined);
      this.currentPointPosition = null;
      this.currentPointId = null;
      this.updateSource();
    },
    // закрыть попап
    closePopup(event) {
      this.overlay.setPosition(undefined);
      event.target.blur();
    },
  },
};
</script>

<style>
/* Попап (стили из примера на openlayers)*/
.ol-popup {
  position: absolute;
  background-color: white;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
  padding: 15px 30px 0 15px;
  border-radius: 4px;
  border: 1px solid #cccccc;
  bottom: 12px;
  left: -50px;
  width: 281px;
}
.ol-popup:after,
.ol-popup:before {
  top: 100%;
  border: solid transparent;
  content: " ";
  height: 0;
  width: 0;
  position: absolute;
  pointer-events: none;
}
.ol-popup:after {
  border-top-color: white;
  border-width: 10px;
  left: 48px;
  margin-left: -10px;
}
.ol-popup:before {
  border-top-color: #cccccc;
  border-width: 11px;
  left: 48px;
  margin-left: -11px;
}
.ol-popup-closer {
  text-decoration: none;
  position: absolute;
  top: 2px;
  right: 8px;
}
.ol-popup-closer:after {
  content: "✖";
}
.ol-popup-content {
  display: grid;
  margin-bottom: -10px;
  gap: 15px;
  grid-template-columns: 1fr 1fr 1fr;
}
</style>