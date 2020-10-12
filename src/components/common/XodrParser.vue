<template>
    <div ref="threeDemo"></div>
</template>

<script>
import * as THREE from 'three';
export default {
    name: 'xodrParser',
    data(){
        return{
            xodrPath:'Town03.xodr',
            xmlDoc:null,
            controls:{
                scene: null,
                camera: null,
                renderer: null
            },
            render:{
                viewControl: null
            }
        }
    },
    created(){
        this.$nextTick(()=>{
            this.init()
        })
    },

    methods:{
        init(){
            //去data里面找数据
            let {xodrPath,controls,xmlDoc} = this;
            //储存一个全局变量
           
            //生成平面
            let planeGeometry = new THREE.PlaneGeometry(60, 20, 10, 10);
            let planeMaterial = new THREE.MeshLambertMaterial({color:0xffffff});
            let plane = new THREE.Mesh(planeGeometry, planeMaterial); //构建地板的材料
            plane.rotation.x = -0.8*Math.PI;
            plane.position.x = 0;
            plane.position.y = 0;
            plane.position.z = 0;
            plane.receiveShadow = true;

            let spotLight = new THREE.SpotLight(0xffffff);
            spotLight.position.set(-10, 10, -10);
            spotLight.castShadow = true;

            this.scene = new THREE.Scene();
            //相机 视场fov 长宽比 近面 远面
            this.camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
            this.camera.position.x = 0;
            this.camera.position.y = 0;
            this.camera.position.z = 500;
            this.camera.lookAt(plane.position);
            this.renderer = new THREE.WebGLRenderer();
            this.scene.add(plane);
            this.scene.add(spotLight);
            this.renderer.setSize(1000,600);
            this.$refs.threeDemo.append(this.renderer.domElement);
            this.xmlDoc = this.loadXML(xodrPath);
            this.drawCurve();
            this.parseXodr();
            this.renderScene();
        },
        loadXML(xmlFile){
            var xmlDoc;
            if (window.ActiveXObject) {
                xmlDoc = new ActiveXObject("Microsoft.XMLDOM");
                xmlDoc.async = false;
                xmlDoc.load(xmlFile);
            }
            else if (document.implementation && document.implementation.createDocument) {
                var xmlObj = new XMLHttpRequest(); //用AJAX中常见的套路来就可以解决了，不影响IE、FIREFOX的原生加载XML 
                xmlObj.overrideMimeType("text/xml");
                xmlObj.open("GET", xmlFile, false);
                xmlObj.send(null);
                xmlDoc = xmlObj.responseXML; 
            } else {
                alert('您的浏览器不支持该系统脚本！');
            }
            return xmlDoc;
        },
        renderScene() {
            let {controls, scene, camera, renderer} = this;
            requestAnimationFrame(this.renderScene);
            renderer.render(scene, camera);
        },
        parseXodr(){
            let {xmlDoc} = this;
            //获取roadNodes的节点
            var roadNodes = this.xmlDoc.childNodes[0].childNodes;
            for(var i=0;i<roadNodes.length;i++){
                //循环找到road的节点
                if(roadNodes[i].nodeName=='road'){
                    this.parseRoad(roadNodes[i])
                }
            }
        },
        parseRoad(roadNode){
            var roadAttributes = roadNode.childNodes;
            for(var i=0;i<roadAttributes.length;i++){
                //找到plainview
                if(roadAttributes[i].nodeName == 'planView'){
                    // 这里处理plainview的内容
                    this.parsePlanView(roadAttributes[i]);
                }else if(roadAttributes[i].nodeName == 'lanes'){
                    this.parseLanes(roadAttributes[i]);
                }
            }
        },
        parsePlanView(planViewNode){
            //这个是road的planview
            var planViewAttributes = planViewNode.childNodes;
            for(var i=0;i<planViewAttributes.length;i++){
                if(planViewAttributes[i].nodeName == 'geometry'){
                    // 这里处理plainview的内容
                    this.parseGeometry(planViewAttributes[i]);
                }
            }
        },
        parseLanes(lanesNode){
            //TODO 处理lanes的内容
            var lanesAttributes = lanesNode.childNodes;
            for(var i=0;i<lanesAttributes.length;i++){
                if(lanesAttributes[i].nodeName == 'laneSection'){
                    // 这里处理plainview的内容 laneoffset lanesection
                    this.parseLaneSection(lanesAttributes[i]);
                }
            }
        },
        parseLaneSection(LaneSectionNode){
            var lanesectionAttributes = LaneSectionNode.childNodes;
        },
        parseGeometry(geometryNode){
            //geometry有多种形式 lane最常见 arc弧度曲线较为常见
            var geometryAttributes = geometryNode.childNodes;
            var x = parseFloat(geometryNode.getAttribute('x'));
            var y = parseFloat(geometryNode.getAttribute('y'));
            var hdg = parseFloat(geometryNode.getAttribute('hdg'));
            var length = parseFloat(geometryNode.getAttribute('length'));
            for(var i=0;i<geometryAttributes.length;i++){
                if(geometryAttributes[i].nodeName != '#text'){
                    // 这里处理plainview的内容
                    if(geometryAttributes[i].nodeName == 'line'){
                        this.drawLine(x, y, hdg, length);
                    }else if(geometryAttributes[i].nodeName == 'arc'){
                        var arc = parseFloat(geometryAttributes[i].getAttribute('curvature'));
                        this.drawCurve(x, y, hdg, length, arc);
                    }else{
                        console.log('this type need handle');
                    }
                }
            }
        },
        drawLine(x, y, hdg, length){
            let{controls,scene} = this;
            var result = [];
            var geometry = new THREE.Geometry();
            var start = new THREE.Vector3(x, y, 0);
            var end = new THREE.Vector3(x+ length*Math.cos(hdg), y+ length*Math.sin(hdg),0);
            var line = new THREE.LineCurve3(start, end);
            var points = line.getPoints(200);
            for(var i =0; i<points.length;i++){
                var x_dis = points[i].x - x;
                var y_dis = points[i].y - y;
                var dis = Math.sqrt(x_dis*x_dis + y_dis*y_dis); 
                result.push([dis, points[i]]);
            }
            geometry.setFromPoints(points);
            var material = new THREE.LineBasicMaterial({color: 0xff0000});
            var threeLine = new THREE.Line(geometry, material);
            this.scene.add(threeLine);
            return result;
        },
        drawCurve(x, y ,hdg, length, arc){
            let{controls,scene} = this;
            //arc > 0 顺时针 arc<0 逆时针呢
            var curve;
            if(arc>0){
                // 正数是逆时针
                var radius = 1.0/arc;
                var angle = length/radius;
                var alpha = hdg - Math.PI/2;
                curve = new THREE.EllipseCurve(
                    x - radius*Math.cos(alpha),  y - radius*Math.sin(alpha),            // ax, aY
                    radius, radius,           // xRadius, yRadius
                    //Math.PI/2-(Math.PI-angle)-hdg, Math.PI/2-hdg,
                    0,  angle,  // aStartAngle, aEndAngle
                    false,            // aClockwise 和arc的正负有关
                    alpha                 // aRotation
                );
            }else{
                // 负数是顺时针
                var radius = 1.0/(-arc);
                var angle = length/radius;
                var alpha = hdg + Math.PI/2;
                curve = new THREE.EllipseCurve(
                    x - radius*Math.cos(alpha),  y - radius*Math.sin(alpha),            // ax, aY
                    radius, radius,           // xRadius, yRadius
                    //Math.PI/2-(Math.PI-angle)-hdg, Math.PI/2-hdg,
                    0,  -angle,  // aStartAngle, aEndAngle
                    true,            // aClockwise 和arc的正负有关
                    alpha                 // aRotation 逆时针
                );
            }
            var points = curve.getPoints(200);
            var geometry = new THREE.BufferGeometry().setFromPoints( points );
            var material = new THREE.LineBasicMaterial( { color : 0xff0000 } );
            // Create the final object to add to the scene
            var ellipse = new THREE.Line(geometry, material);
            this.scene.add(ellipse);
        }
    }
    
}
</script>

