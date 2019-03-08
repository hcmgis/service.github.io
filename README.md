# KHAI THÁC DỊCH VỤ HCMGIS MAPS

Khai thác dịch vụ HCMGIS Maps thông qua WMS
-------------
**Demo**
 - https://hcmgis.github.io/service.github.io
-------------
**Hướng dẫn**
Đính kèm file Leaflet CSS vào phần head html
```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.3.4/dist/leaflet.css"
      integrity="sha512-puBpdR0798OZvTTbP4A8Ix/l+A4dHDD0DGqYW6RQ+9jxkRFclaxxQb/SJAWZfWAkuyeQUytO7+7N4QKrDh+drA=="
      crossorigin=""/>
```
Đính kèm file Leaflet JS vào sau dòng Leaflet CSS
```html
<script src="https://unpkg.com/leaflet@1.3.4/dist/leaflet.js"
        integrity="sha512-nMMmRyTVoLYqjP9hrbed9S+FzjZHW5gY1TWCHA5ckwXZBadntCNs8kEqAWdrb9O7rxbCaA4lKTIWjDXZxflOcA=="
        crossorigin=""></script>
```
Đặt thẻ div với id là tên bản đồ
```html
Đặt thẻ div với id là tên bản đồ
<div id='map'></div>
```
Thiết lập style CSS cho map container 
```css
#map { position:absolute; top:0; bottom:0; width:100%; }
```
Tại khối script js, ta dựng bản đồ như sau:
- Khởi tạo bản đồ
```js
let map = L.map('map').setView([10.800218, 106.662294], 10);
```
- Tạo lớp nền và lớp WMS (L.tileLayer) với 2 tham số đầu vào: đường dẫn url và tùy chọn
```html
let baseLayer = L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}', {
    attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
    maxZoom: 18,
    id: 'mapbox.streets',
    accessToken: 'pk.eyJ1IjoidHR1bmdibXQiLCJhIjoiY2EzNDFhZjU4ZThkNzY5NTU3M2U1YWFiNmY4OTE3OWQifQ.Bo1ss5J4UjPPOjmq9S3VQw'
});

let wmsLayer = L.tileLayer.wms('https://maps.hcmgis.vn/geoserver/gwc/service/wms', {
    layers: 'hcm_map:hcm_map_all'
});
```
- Thêm các lớp đã tạo vào bản đồ
```js
let basemaps = {
    Mapbox: baseLayer.addTo(map),
    'HCMGIS Maps': wmsLayer,
};

L.control.layers(basemaps).addTo(map);
```
- Để lấy thêm thông tin thuộc tính vào từ WMS, người dùng thay L.tileLayer.wms bằng L. tileLayer.betterWms
```js
L.TileLayer.BetterWMS = L.TileLayer.WMS.extend({

    onAdd: function (map) {
        // Triggered when the layer is added to a map.
        //   Register a click listener, then do all the upstream WMS things
        L.TileLayer.WMS.prototype.onAdd.call(this, map);
        map.on('click', this.getFeatureInfo, this);
    },

    onRemove: function (map) {
        // Triggered when the layer is removed from a map.
        //   Unregister a click listener, then do all the upstream WMS things
        L.TileLayer.WMS.prototype.onRemove.call(this, map);
        map.off('click', this.getFeatureInfo, this);
    },

    getFeatureInfo: function (evt) {
        // Make an AJAX request to the server and hope for the best
        var url = this.getFeatureInfoUrl(evt.latlng),
            showResults = L.Util.bind(this.showGetFeatureInfo, this);
        $.ajax({
            url: url,
            success: function (data, status, xhr) {
                var err = typeof data === 'string' ? null : data;
                showResults(err, evt.latlng, data);
            },
            error: function (xhr, status, error) {
                showResults(error);
            }
        });
    },

    getFeatureInfoUrl: function (latlng) {
        // Construct a GetFeatureInfo request URL given a point
        var point = this._map.latLngToContainerPoint(latlng, this._map.getZoom()),
            size = this._map.getSize(),

            params = {
                request: 'GetFeatureInfo',
                service: 'WMS',
                srs: 'EPSG:4326',
                styles: this.wmsParams.styles,
                transparent: this.wmsParams.transparent,
                version: this.wmsParams.version,
                format: this.wmsParams.format,
                bbox: this._map.getBounds().toBBoxString(),
                height: size.y,
                width: size.x,
                layers: this.wmsParams.layers,
                query_layers: this.wmsParams.layers,
                info_format: 'text/html'
            };

        params[params.version === '1.3.0' ? 'i' : 'x'] = point.x;
        params[params.version === '1.3.0' ? 'j' : 'y'] = point.y;

        return this._url + L.Util.getParamString(params, this._url, true);
    },

    showGetFeatureInfo: function (err, latlng, content) {
        if (err) { console.log(err); return; } // do nothing if there's an error

        // Otherwise show the content in a popup, or something.
        L.popup({ maxWidth: 800})
            .setLatLng(latlng)
            .setContent(content)
            .openOn(this._map);
    }
});

L.tileLayer.betterWms = function (url, options) {
    return new L.TileLayer.BetterWMS(url, options);
};
```
Chèn ảnh hàng không HCMGIS
-------------
Người dùng dựng lớp tileLayer ảnh hàng không và thêm vào bản đồ
```js
let tileLayer = L.tileLayer(
    'http://trueortho.hcmgis.vn/basemap/cache_lidar/{z}/{x}/{y}.jpg', {
        attribution: 'Map data &copy; <a href="https://hcmgis.vn/">HCMGIS</a>'
}).addTo(map);

```

