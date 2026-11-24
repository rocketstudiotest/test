<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MyCione Online - Proceso de Compra Unificado B2B</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background-color: #f4f5f8; overflow-y: auto; }
        .bg-cione-header { background: linear-gradient(90deg, #1e1b4b 0%, #2e2a85 45%, #3b37a8 100%); }
        .bg-cione-sidebar-brand { background: linear-gradient(135deg, #24196b 0%, #372d9c 100%); }
        
        .crosshair-container { position: relative; width: 100%; height: 100%; min-height: 250px; margin: 0 auto; background: #fff; }
        .crosshair-v { position: absolute; width: 1px; height: 100%; background: #9ca3af; left: 50%; top: 0; }
        .crosshair-h { position: absolute; width: 100%; height: 1px; background: #9ca3af; top: 50%; left: 0; }
        
        ::-webkit-scrollbar { width: 7px; height: 7px; }
        ::-webkit-scrollbar-track { background: #f1f5f9; border-radius: 4px; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        
        .accordion-content { transition: max-height 0.3s ease-in-out, opacity 0.3s ease-in-out; overflow: hidden; }
        
        /* Simulación Ventana Windows (Trazador) */
        .win-window { border: 1px solid #ccc; box-shadow: 0 10px 25px rgba(0,0,0,0.2); background: #f0f0f0; border-radius: 6px; overflow: hidden; display: flex; flex-direction: column; }
        .win-titlebar { background: #fff; padding: 6px 12px; font-size: 11px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #ddd; }
        .win-menu { background: #fff; padding: 4px 12px; font-size: 11px; border-bottom: 1px solid #ddd; display: flex; gap: 12px; }
        .win-content { flex: 1; background: #fff; position: relative; display: flex; }
        .win-popup { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #f0f0f0; border: 1px solid #999; box-shadow: 2px 2px 10px rgba(0,0,0,0.2); border-radius: 4px; width: 260px; z-index: 10; }
        .animate-fade-in { animation: fadeIn 0.25s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(4px); } to { opacity: 1; transform: translateY(0); } }
        
        /* Ocultar barra de scroll en las columnas de LC para diseño limpio */
        .custom-scrollbar::-webkit-scrollbar { width: 4px; height: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #e2e8f0; border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #cbd5e1; }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useEffect } = React;

        const MOCK_LENSES = [
            { id: 'p1', familia: 'Monofocales', producto: 'Monofocales de stock', ref: 'CIO-101', fab: 'Cione', precio: '12,50', descuento: '5 % descuento', eciones: '2 €C', tag: 'PRODUCTO PROPIO CIONE', badges: ['Top ventas'], desc: 'Disponibilidad inmediata con tratamientos antirreflejantes multicapa estándar.', mat_tec: 'Orgánico CR-39 con endurecido superficial', clinico: 'Ideal para graduaciones esféricas bajas y medias sin astigmatismos complejos.' },
            { id: 'p2', familia: 'Monofocales', producto: 'Monofocales SV Cione Essilor', ref: 'CIO-102', fab: 'Essilor', precio: '18,90', descuento: '5 % descuento', eciones: '4 €C', tag: 'PRODUCTO PROPIO CIONE', desc: 'Geometría optimizada Cione y precisión Essilor para eliminar aberraciones.', mat_tec: 'Polímero de alto índice Essilor asférico', clinico: 'Minimiza la distorsión periférica y mejora estética en altas graduaciones.' },
            { id: 'p3', familia: 'Monofocales', producto: 'Monofocales SV Cione FF Hoya', ref: 'CIO-103', fab: 'Hoya', precio: '22,40', descuento: '5 % descuento', eciones: '4 €C', tag: 'PRODUCTO PROPIO CIONE', badges: ['Descartado'], desc: 'Tallado Freeform punto a punto que maximiza el campo visual sin distorsión.', mat_tec: 'Material Hoya PNX 1.53 ultra-resistente', clinico: 'Alta resistencia al impacto, recomendada para uso activo y monturas al aire.' },
            { id: 'p4', familia: 'Monofocales', producto: 'Monofocales SV Cione ECO FF', ref: 'CIO-104', fab: 'Cione', precio: '15,00', descuento: '5 % descuento', eciones: '2 €C', tag: 'PRODUCTO PROPIO CIONE', desc: 'Diseño monofocal de entrada que equilibra personalización y coste contenido.', mat_tec: 'Orgánico básico 1.50 tallado digital', clinico: 'Solución económica de alta fiabilidad para ametropías sencillas.' },
            { id: 'p5', familia: 'Monofocales', producto: 'Monofocales minerales', ref: 'CIO-105', fab: 'Cione', precio: '19,50', descuento: '5 % descuento', eciones: '3 €C', tag: 'PRODUCTO PROPIO CIONE', desc: 'Material en vidrio mineral de alto índice para la máxima resistencia al rayado.', mat_tec: 'Cristal mineral Crown 1.52 / 1.60', clinico: 'Máxima durabilidad en entornos de trabajo abrasivos.' },
            { id: 'p6', familia: 'Monofocales', producto: 'Monofocales Cione Mykids', ref: 'CIO-106', fab: 'Cione', precio: '29,90', descuento: '5 % descuento', eciones: '5 €C', tag: 'PRODUCTO PROPIO CIONE', badges: ['Control miopía', 'Novedad'], desc: 'Desenfoque periférico asimétrico para frenar la elongación axial infantil.', mat_tec: 'Policarbonato 1.59 irrompible con filtro UV', clinico: 'Eficacia del 59% en la retención de progresión miópica pediátrica.' },
            
            { id: 'p7', familia: 'Progresivos', producto: 'Progresivos First by Essilor', ref: 'CIO-201', fab: 'Essilor', precio: '24,50', descuento: '5 % descuento', eciones: '4 €C', tag: 'PRODUCTO PROPIO CIONE', desc: 'Transición suave y controlada, ideal para nuevos présbitas.', mat_tec: 'Diseño blando de pasillo largo', clinico: 'Reduce el periodo de neuroadaptación inicial en présbitas prematuros.' },
            { id: 'p8', familia: 'Progresivos', producto: 'Progresivos First by Carl Zeiss', ref: 'CIO-202', fab: 'Zeiss', precio: '26,00', descuento: '5 % descuento', eciones: '4 €C', tag: 'PRODUCTO PROPIO CIONE', badges: ['Top ventas'], desc: 'Precisión ZEISS en un progresivo que minimiza el efecto de balanceo.', mat_tec: 'Tecnología Precision Pure Zeiss', clinico: 'Amplia la visión intermedia para uso simultáneo de múltiples dispositivos.' },
            { id: 'p10', familia: 'Progresivos', producto: 'Progresivos Cione 3.0 by Essilor', ref: 'CIO-204', fab: 'Essilor', precio: '28,90', descuento: '5 % descuento', eciones: '5 €C', tag: 'PRODUCTO PROPIO CIONE', desc: 'Alta resolución con campos ampliados y optimización binocular.', mat_tec: 'Cálculo W.A.V.E. 2.0 y tallado reticular', clinico: 'Visión de alto contraste y mapeo pupilar para baja iluminación.' },
            { id: 'p13', familia: 'Progresivos', producto: 'Progresivos Cione VR by Indo', ref: 'CIO-207', fab: 'Indo', precio: '29,50', descuento: '5 % descuento', eciones: '5 €C', tag: 'PRODUCTO PROPIO CIONE', badges: ['Novedad'], desc: 'Personalización extrema (RV y parámetros faciales) para un confort absoluto.', mat_tec: 'Tecnología FreeMax Indo', clinico: 'Campos 30% más amplios considerando el centro de rotación ocular exacto.' },
            
            // TERCEROS
            { id: 't1', familia: 'Monofocales', producto: 'Monofocales de stock ZEISS', ref: 'EXT-101', fab: 'Zeiss', precio: '19,00', descuento: '10 % descuento', eciones: '2 €C', tag: 'TERCEROS', desc: 'Precisión ZEISS con protección total UVProtect hasta 400 nm de serie.', mat_tec: 'Material transparente ZEISS con UVProtect', clinico: 'Protección ultravioleta a nivel de gafa de sol en lente transparente diaria.' },
            { id: 't2', familia: 'Monofocales', producto: 'Monofocales ClearView de stock', ref: 'EXT-102', fab: 'Zeiss', precio: '24,00', descuento: '10 % descuento', eciones: '3 €C', tag: 'TERCEROS', badges: ['Top ventas'], desc: 'Diseño en 700 puntos para una lente más plana, fina y con óptica tres veces más amplia.', mat_tec: 'Optimización de 700 cuadrantes asféricos', clinico: 'Mejora estética en graduaciones altas (lente más plana y ligera).' },
            { id: 't4', familia: 'Monofocales', producto: 'Monofocales Eyezen Boost', ref: 'EXT-104', fab: 'Essilor', precio: '27,50', descuento: '10 % descuento', eciones: '4 €C', tag: 'TERCEROS', desc: 'Filtro Eye Protect y DualOptim para aliviar severamente la fatiga digital.', mat_tec: 'Corte UV-azul selectivo de Essilor', clinico: 'Adición inferior (+0.4D / +0.6D) para relajar la acomodación en pantallas.' },
            { id: 't5', familia: 'Monofocales', producto: 'MiYOSMART Spectacles', ref: 'EXT-105', fab: 'Hoya', precio: '29,90', descuento: '10 % descuento', eciones: '4 €C', tag: 'TERCEROS', badges: ['Control miopía', 'Varios colores'], desc: 'Tecnología patentada D.I.M.S. que frena clínicamente la miopía en niños en un 60%.', mat_tec: 'Zona central nítida y cientos de micro-islas de desenfoque', clinico: 'Tratamiento gold-standard no invasivo en ralentización axial pediátrica.' },
            { id: 't6', familia: 'Progresivos', producto: 'Progresivos SmartLife', ref: 'EXT-201', fab: 'Zeiss', precio: '29,50', descuento: '10 % descuento', eciones: '4 €C', tag: 'TERCEROS', badges: ['Novedad'], desc: 'Tecnología SmartView para un enfoque dinámico en un mundo conectado.', mat_tec: 'Adaptación a la edad visual y tamaño de pupila', clinico: 'Responde a los cambios rápidos de mirada del smartphone al horizonte.' },
            { id: 't8', familia: 'Progresivos', producto: 'Varilux Comfort Max', ref: 'EXT-203', fab: 'Essilor', precio: '29,90', descuento: '10 % descuento', eciones: '4 €C', tag: 'TERCEROS', desc: 'Flexibilidad postural máxima para encontrar la zona de visión útil sin posturas rígidas.', mat_tec: 'Flexoptim Technology para ampliar corredor', clinico: 'Expande el área de visión útil para reducir dolores cervicales y posturales.' }
        ];

        // Añadir defaults a los que no tienen
        MOCK_LENSES.forEach(l => {
            if(!l.mat_tec) l.mat_tec = 'Material y diseño avanzado optimizado para la gama.';
            if(!l.clinico) l.clinico = 'Mejora de agudeza visual y adaptabilidad inmediata en condiciones reales.';
        });

        const BASE_LC_CATALOG = [
            // PRODUCTO PROPIO (CIONE BASIC)
            { cat_propio: 'Cione Basic', prop: true, reemplazo: 'Diarias', material: 'Hidrogel', tipo: 'Miopía/Hipermetropía', fab: 'Clarlab', promo: 'Descuento por pack', precio: '12,50', descuento: '10 % descuento', eciones: '2 €C', nombre: 'Cione Basic Diaria 30 uds', ref: 'LC-101', formato: 'Caja 30 lentes', badges: ['Top ventas'], desc: 'Hidrogel biocompatible de alta hidratación para uso diario cómodo.', mat_tec: 'Polímero HEMA clásico (55% agua)', clinico: 'Buena humectabilidad inicial para porte < 8 horas diarias.' },
            { cat_propio: 'Cione Basic', prop: true, reemplazo: 'Mensuales', material: 'Hidrogel', tipo: 'Miopía/Hipermetropía', fab: 'Clarlab', promo: null, precio: '9,20', descuento: '5 % descuento', eciones: '2 €C', nombre: 'Cione Basic Mensual 3 uds', ref: 'LC-202', formato: 'Caja 3 lentes', badges: ['Descartado'], desc: 'Formato 3 lentes en hidrogel de alta tolerancia.', mat_tec: 'Material no iónico resistente', clinico: 'Resistencia probada a depósitos proteicos durante 30 días.' },
            
            // PRODUCTO PROPIO (CIONE PREMIUM)
            { cat_propio: 'Cione Premium', prop: true, reemplazo: 'Diarias', material: 'Silicona', tipo: 'Miopía/Hipermetropía', fab: 'CooperVision', promo: 'Descuento por pack', precio: '15,90', descuento: '10 % descuento', eciones: '3 €C', nombre: 'Cione Premium SiHy 30 uds', ref: 'LC-103', formato: 'Caja 30 lentes', desc: 'Hidrogel de silicona de 3ª generación con alta transmisibilidad al oxígeno.', mat_tec: 'Silicona de bajo módulo con filtro UV', clinico: 'Evita la hipoxia corneal permitiendo un uso seguro y prolongado.' },
            { cat_propio: 'Cione Premium', prop: true, reemplazo: 'Mensuales', material: 'Silicona', tipo: 'Astigmatismo', fab: 'Menicom', promo: 'Descuento por pack', precio: '20,50', descuento: '10 % descuento', eciones: '4 €C', nombre: 'Cione Premium Toric SiHy 6 uds', ref: 'LC-205', formato: 'Caja 6 lentes', badges: ['Top ventas'], desc: 'Superficie ultralisada de baja fricción para porte continuo mensual.', mat_tec: 'Tecnología Nanogloss de ultra baja fricción', clinico: 'Estabilización dinámica por parpadeo para agudeza visual constante.' },
            { cat_propio: 'Cione Premium', prop: true, reemplazo: 'Diarias', material: 'Hidrogel', tipo: 'Miopía/Hipermetropía', fab: 'CooperVision', promo: null, precio: '28,50', descuento: '5 % descuento', eciones: '4 €C', nombre: 'Cione MyKids Myopia Control 30 uds', ref: 'LC-106', formato: 'Caja 30 lentes', badges: ['Control miopía', 'Novedad'], desc: 'Óptica de desenfoque periférico para ralentizar la progresión de la miopía.', mat_tec: 'Diseño óptico dual-focus (terapéutico)', clinico: 'Indicado para niños a partir de 8 años con rápida evolución miópica.' },
            
            // PRODUCTO PROPIO (CIONE ADVANCED)
            { cat_propio: 'Cione Advanced', prop: true, reemplazo: 'Mensuales', material: 'Silicona', tipo: 'Multifocal', fab: 'Mark\'ennovy', promo: null, precio: '36,50', descuento: '5 % descuento', eciones: '5 €C', nombre: 'Cione Advanced Custom Progresiva 3 uds', ref: 'LC-217', formato: 'Caja 3 lentes', desc: 'Personalización de diámetro pupilar y adición en silicona.', mat_tec: 'Torneado micrométrico 100% a medida', clinico: 'Soluciona adaptaciones complejas de presbicia y queratometrías extremas.' },
            { cat_propio: 'Cione Advanced', prop: true, reemplazo: 'Cosméticas', material: 'Hidrogel', tipo: 'Cosmética', fab: 'Shine', promo: 'Descuento por pack', precio: '19,90', descuento: '10 % descuento', eciones: '3 €C', nombre: 'Cione Color Natural 2 uds', ref: 'LC-401', formato: 'Caja 2 lentes', badges: ['Varios colores'], desc: 'Lente cosmética Cione con pigmentación natural y confort diario.', mat_tec: 'Matriz de color tricapa encapsulada', clinico: 'Seguridad máxima y prevención de toxicidad al no haber contacto del pigmento.' },

            // TERCEROS - Alcon
            { prop: false, reemplazo: 'Diarias', material: 'Silicona', tipo: 'Miopía/Hipermetropía', fab: 'Alcon', promo: 'Descuento por pack', precio: '22,50', descuento: '12 % descuento', eciones: '3 €C', nombre: 'Dailies Total1 (30 uds)', ref: 'EXT-101', formato: 'Caja 30 lentes', badges: ['Top ventas'], desc: 'Lente con gradiente acuoso que se acerca al 100% de agua en la superficie.', mat_tec: 'Delefilcon A (Gradiente de agua patentado)', clinico: 'El confort se mantiene intacto hasta 16 horas, ideal para ojo seco.' },
            { prop: false, reemplazo: 'Mensuales', material: 'Silicona', tipo: 'Astigmatismo', fab: 'Alcon', promo: null, precio: '28,50', descuento: '8 % descuento', eciones: '3 €C', nombre: 'Air Optix Plus Hydraglyde Toric (6 uds)', ref: 'EXT-202', formato: 'Caja 6 lentes', desc: 'Estabilidad excepcional con tecnología matriz de humectación Hydraglyde.', mat_tec: 'Lotrafilcon B con matriz humectante', clinico: 'Evita la sequedad inducida al final del ciclo de porte de 30 días.' },

            // TERCEROS - CooperVision
            { prop: false, reemplazo: 'Diarias', material: 'Silicona', tipo: 'Miopía/Hipermetropía', fab: 'CooperVision', promo: 'Descuento por pack', precio: '18,90', descuento: '10 % descuento', eciones: '2 €C', nombre: 'Clariti 1 day (30 uds)', ref: 'EXT-103', formato: 'Caja 30 lentes', desc: 'Hidrogel de silicona diario con filtro UV y tecnología WetLoc natural.', mat_tec: 'Somofilcon A con WetLoc', clinico: 'Transición natural y económicamente viable desde hidrogel a silicona.' },
            { prop: false, reemplazo: 'Diarias', material: 'Hidrogel', tipo: 'Miopía/Hipermetropía', fab: 'CooperVision', promo: null, precio: '36,00', descuento: '8 % descuento', eciones: '4 €C', nombre: 'MiSight 1 day (30 uds)', ref: 'EXT-109', formato: 'Caja 30 lentes', badges: ['Control miopía', 'Top ventas'], desc: 'Tecnología ActivControl clínicamente probada para frenar la miopía infantil.', mat_tec: 'Omafilcon A (Diseño dual-focus FDA)', clinico: 'Tratamiento gold-standard homologado por la FDA para control miópico.' },
            
            // TERCEROS - Bausch & Lomb
            { prop: false, reemplazo: 'Diarias', material: 'Silicona', tipo: 'Miopía/Hipermetropía', fab: 'Bausch & Lomb', promo: 'Descuento por pack', precio: '21,50', descuento: '8 % descuento', eciones: '3 €C', nombre: 'Biotrue ONEday (30 uds)', ref: 'EXT-110', formato: 'Caja 30 lentes', desc: 'Inspiradas en la biología del ojo para igualar la hidratación natural de la córnea.', mat_tec: 'Nesofilcon A (78% de agua)', clinico: 'Mantiene el 98% de su hidratación durante todo el día sin gotas adicionales.' },
            { prop: false, reemplazo: 'Mensuales', material: 'Silicona', tipo: 'Multifocal', fab: 'Bausch & Lomb', promo: 'Descuento por pack', precio: '35,00', descuento: '10 % descuento', eciones: '4 €C', nombre: 'Ultra for Presbyopia (6 uds)', ref: 'EXT-206', formato: 'Caja 6 lentes', desc: 'Diseño 3-Zone Progressive para transiciones perfectas a cualquier distancia.', mat_tec: 'Samfilcon A con MoistureSeal', clinico: 'Ideal para usuarios de múltiples pantallas digitales con presbicia moderada.' },

            // TERCEROS - Johnson & Johnson
            { prop: false, reemplazo: 'Diarias', material: 'Silicona', tipo: 'Astigmatismo', fab: 'Johnson & Johnson', promo: 'Descuento por pack', precio: '29,00', descuento: '10 % descuento', eciones: '4 €C', nombre: 'Acuvue Oasys 1-Day Astigmatism', ref: 'EXT-112', formato: 'Caja 30 lentes', badges: ['Novedad'], desc: 'Estabilización acelerada por el parpadeo para visión siempre nítida.', mat_tec: 'Senofilcon A con tecnología HydraLuxe', clinico: 'Trabaja conjuntamente con la película lagrimal para reducir fricción palpebral.' },

            // TERCEROS - Mark'ennovy
            { prop: false, reemplazo: 'Mensuales', material: 'Hidrogel', tipo: 'Multifocal tórica', fab: 'Mark\'ennovy', promo: null, precio: '44,50', descuento: '8 % descuento', eciones: '5 €C', nombre: 'Gentle 80 Multifocal Toric (3 uds)', ref: 'EXT-208', formato: 'Caja 3 lentes', badges: ['Varios colores'], desc: 'Hidrogel biomimético con módulo de elasticidad similar al tejido corneal humano.', mat_tec: 'Filcon V con 80% de hidratación', clinico: 'Diseño ultra-personalizado hecho a medida para morfologías corneales atípicas.' }
        ];

        // Rellenar dinámicamente hasta 43 y 72 para cumplir requerimiento de cuenta
        const FULL_LC_CATALOG = [];
        const baseProp = BASE_LC_CATALOG.filter(x => x.prop);
        const baseTerc = BASE_LC_CATALOG.filter(x => !x.prop);
        
        FULL_LC_CATALOG.push(...baseProp);
        for(let i = baseProp.length; i < 43; i++) {
            let clone = {...baseProp[i % baseProp.length]};
            clone.id = 'lcp_pad_' + i;
            FULL_LC_CATALOG.push(clone);
        }
        
        FULL_LC_CATALOG.push(...baseTerc);
        for(let i = baseTerc.length; i < 72; i++) {
            let clone = {...baseTerc[i % baseTerc.length]};
            clone.id = 'lct_pad_' + i;
            FULL_LC_CATALOG.push(clone);
        }

        const DISEÑOS = ["CIONE Advanced 3.0 Essilor", "CIONE Advanced 3.0 Polarizado", "CIONE Basic Essilor", "CIONE Elite 2.0 Essilor"];
        const INDICES = ["1.50 Orgánico", "1.60 Orgánico", "1.67 Orgánico", "1.74 Orgánico", "1.52 Mineral"];
        const TRATAMIENTOS = ["Anirreflejante Cl", "Anti-Reflex", "Anti-Reflex Blue", "Anti-Reflex Max"];

        const parsePrice = (priceStr) => parseFloat(String(priceStr || "0").replace(',', '.')) || 0;
        const formatPrice = (priceNum) => (Number(priceNum) || 0).toFixed(2).replace('.', ',');

        const getFamilyIcon = (title) => {
            switch(title) {
                case 'Monofocales': return <svg className="w-[18px] h-[18px] text-indigo-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10" strokeWidth="1.8"/><circle cx="12" cy="12" r="3" strokeWidth="1.8"/></svg>;
                case 'Progresivos': return <svg className="w-[18px] h-[18px] text-indigo-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M3 12h18" /><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M12 3v18" opacity="0.3"/><circle cx="12" cy="12" r="10" strokeWidth="1.8"/></svg>;
                case 'Ocupacionales': return <svg className="w-[18px] h-[18px] text-indigo-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><rect x="2" y="4" width="20" height="13" rx="2" strokeWidth="1.8"/><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M8 21h8m-4-4v4"/></svg>;
                case 'Bifocales': return <svg className="w-[18px] h-[18px] text-indigo-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10" strokeWidth="1.8"/><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M4 16h16"/></svg>;
                default: return <svg className="w-[17px] h-[17px] text-indigo-700" fill="none" stroke="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="9" strokeWidth="1.8"/></svg>;
            }
        };

        const getBadgeCls = (b) => {
            if (b === 'Novedad') return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-teal-100 text-teal-800 border border-teal-200';
            if (b === 'Control miopía') return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-purple-100 text-purple-800 border border-purple-200';
            if (b === 'Top ventas') return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-amber-100 text-amber-800 border border-amber-300';
            if (b === 'Descartado') return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-gray-200 text-gray-500 border border-gray-300';
            if (b === 'Varios colores') return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-pink-100 text-pink-800 border border-pink-200';
            return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-red-50 text-red-700 border border-red-200';
        };

        const getPromoCls = (p) => {
            if (p === 'Descuento por pack' || p === 'Descuento por stock') return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-blue-600 text-white';
            return 'text-[8px] font-bold px-1.5 py-0.5 rounded bg-emerald-600 text-white';
        };

        function App() {
            const [activeCategory, setActiveCategory] = useState('inicio');
            const [isSidebarCollapsed, setIsSidebarCollapsed] = useState(true);
            const [modalOpen, setModalOpen] = useState(false);
            const [selectedLens, setSelectedLens] = useState(null);
            const [favorites, setFavorites] = useState({ p1: true, lcp_d1: true });
            
            // Cart state
            const [cartItems, setCartItems] = useState([]);
            const [showCart, setShowCart] = useState(false);
            
            // Asistente Lentes Graduadas
            const [asistenteProv, setAsistenteProv] = useState([]);
            const [asistenteOnlyFavs, setAsistenteOnlyFavs] = useState(false);
            const [asistenteGrad, setAsistenteGrad] = useState({ odEsf: '', odCil: '', odEje: '', odAdd: '', oiEsf: '', oiCil: '', oiEje: '', oiAdd: '' });
            const [expandedDetails, setExpandedDetails] = useState({});

            // Modal Config Lentes Graduadas
            const [ojoMode, setOjoMode] = useState('ambos');
            const [modalSpecs, setModalSpecs] = useState({ odDiseno: '', odIndice: '', odTratamiento: '', oiDiseno: '', oiIndice: '', oiTratamiento: '' });
            const [modalGrad, setModalGrad] = useState({ odEsf: '', odCil: '', odEje: '', odAdd: '', oiEsf: '', oiCil: '', oiEje: '', oiAdd: '' });

            // Asistente Lentes Contacto
            const [lcTab, setLcTab] = useState('asistente');
            const [lcSearchFilters, setLcSearchFilters] = useState({ reemplazo: 'Todas', material: 'Todos', tipo: 'Todos', fabricante: 'Todos', catalogo: 'Todo', promociones: 'Todas', soloFavoritos: false });
            const [lcGrad, setLcGrad] = useState({ odEsf: '', odCil: '', odEje: '', odAdd: '', odRadio: '', odDiam: '', oiEsf: '', oiCil: '', oiEje: '', oiAdd: '', oiRadio: '', oiDiam: '' });
            const [lcExpandedDetails, setLcExpandedDetails] = useState({});
            
            // Configurar Lente Contacto Modal
            const [lcModalOpen, setLcModalOpen] = useState(false);
            const [selectedLc, setSelectedLc] = useState(null);
            const [lcBoxes, setLcBoxes] = useState({ od: 1, oi: 1 });

            const toggleFav = (id) => setFavorites(prev => ({ ...prev, [id]: !prev[id] }));
            const toggleDetail = (id, isLc = false) => {
                if (isLc) setLcExpandedDetails(prev => ({ ...prev, [id]: !prev[id] }));
                else setExpandedDetails(prev => ({ ...prev, [id]: !prev[id] }));
            };

            const toggleAsistenteProv = (p) => setAsistenteProv(prev => prev.includes(p) ? prev.filter(x => x !== p) : [...prev, p]);

            const openConfig = (lens, prefillGrad = null) => {
                setSelectedLens(lens);
                if (prefillGrad) setModalGrad({ ...prefillGrad });
                else setModalGrad({ odEsf: '', odCil: '', odEje: '', odAdd: '', oiEsf: '', oiCil: '', oiEje: '', oiAdd: '' });
                setModalSpecs({ odDiseno: '', odIndice: '', odTratamiento: '', oiDiseno: '', oiIndice: '', oiTratamiento: '' });
                setOjoMode('ambos');
                setModalOpen(true);
            };

            const copyToLeft = (type) => {
                if (type === 'grad') {
                    setAsistenteGrad(prev => ({ ...prev, oiEsf: prev.odEsf, oiCil: prev.odCil, oiEje: prev.odEje, oiAdd: prev.odAdd }));
                } else if (type === 'lc') {
                    setLcGrad(prev => ({ ...prev, oiEsf: prev.odEsf, oiCil: prev.odCil, oiEje: prev.odEje, oiAdd: prev.odAdd, oiRadio: prev.odRadio, oiDiam: prev.odDiam }));
                }
            };
            const copyToRight = (type) => {
                if (type === 'grad') {
                    setAsistenteGrad(prev => ({ ...prev, odEsf: prev.oiEsf, odCil: prev.oiCil, odEje: prev.oiEje, odAdd: prev.oiAdd }));
                } else if (type === 'lc') {
                    setLcGrad(prev => ({ ...prev, odEsf: prev.oiEsf, odCil: prev.oiCil, odEje: prev.oiEje, odAdd: prev.oiAdd, odRadio: prev.oiRadio, odDiam: prev.oiDiam }));
                }
            };

            // Filtrado Lentes Graduadas
            const propPropios = MOCK_LENSES.filter(l => l.tag === 'PRODUCTO PROPIO CIONE');
            const propTerceros = MOCK_LENSES.filter(l => l.tag === 'TERCEROS');
            const hasAdditionEntered = Boolean(asistenteGrad.odAdd.trim() || asistenteGrad.oiAdd.trim());

            // Filtrado Lentes de Contacto
            const hasLcAddition = Boolean(lcGrad.odAdd.trim() || lcGrad.oiAdd.trim());
            const hasLcCylinder = Boolean(lcGrad.odCil.trim() || lcGrad.oiCil.trim());
            
            const matchesLcItem = (lens) => {
                if (lcSearchFilters.reemplazo !== 'Todas' && lens.reemplazo !== lcSearchFilters.reemplazo) return false;
                if (lcSearchFilters.catalogo === 'Producto propio' && !lens.prop) return false;
                if (lcSearchFilters.catalogo === 'Terceros' && lens.prop) return false;
                if (hasLcAddition && !lens.tipo.toLowerCase().includes('multifocal')) return false;
                if (hasLcCylinder && !lens.tipo.toLowerCase().includes('astigmatismo') && !lens.tipo.toLowerCase().includes('tórica')) return false;
                return true;
            };

            const renderBullets = (item) => (
                <ul className="list-disc pl-4 mt-2 space-y-1.5 text-[11px] text-gray-700">
                    <li><strong className="text-gray-900">Especificación destacada:</strong> {item.desc}</li>
                    <li><strong className="text-gray-900">Material y Tecnología:</strong> {item.mat_tec}</li>
                    <li><strong className="text-gray-900">Beneficio clínico:</strong> {item.clinico}</li>
                </ul>
            );

            const renderAsistenteGroup = (title, items) => {
                if (items.length === 0) return null;
                
                // Aplicar filtros proveedor/fav de la barra
                let filteredItems = items.filter(lens => {
                    const isSpecific = ['Essilor', 'Hoya', 'Zeiss', 'Indo', 'Rodenstock'].includes(lens.fab);
                    const matchesOtros = asistenteProv.includes('Otros') && !isSpecific;
                    const matchesProv = asistenteProv.length === 0 || asistenteProv.includes(lens.fab) || matchesOtros;
                    const matchesFav = !asistenteOnlyFavs || !!favorites[lens.id];
                    const matchesAdd = !hasAdditionEntered || lens.familia !== 'Monofocales';
                    return matchesProv && matchesFav && matchesAdd;
                });

                return (
                    <div className="border border-gray-200 rounded-lg p-2.5 bg-white shadow-sm flex flex-col h-full min-w-[300px]">
                        <h3 className="font-bold text-gray-900 text-[13px] mb-2 pb-1.5 border-b border-gray-100 flex items-center gap-1.5">
                            {getFamilyIcon(title)} {title} <span className="text-gray-400 text-[10px] font-normal ml-0.5">({filteredItems.length})</span>
                        </h3>
                        <div className="flex flex-col gap-1 flex-1">
                            {filteredItems.length === 0 && <div className="text-xs text-gray-400 p-2 text-center">Sin resultados</div>}
                            {filteredItems.map(lens => {
                                const isDetailOpen = !!expandedDetails[lens.id];
                                return (
                                    <div key={lens.id} className="border border-gray-200 rounded-md mb-1.5 overflow-hidden">
                                        <div onClick={() => openConfig(lens, asistenteGrad)} className="flex justify-between items-center py-2 px-2 hover:bg-indigo-50/50 cursor-pointer text-gray-800 transition-colors">
                                            <div className="flex flex-col pr-2">
                                                <div className="flex items-center gap-2 flex-wrap mb-1">
                                                    <span className="text-[11px] font-bold">{lens.producto}</span>
                                                    {lens.badges && lens.badges.map(b => <span key={b} className={getBadgeCls(b)}>{b}</span>)}
                                                    {lens.promo && <span className={getPromoCls(lens.promo)}>{lens.promo}</span>}
                                                </div>
                                                <div className="text-[10px] text-gray-500">{lens.fab} · {lens.precio}€</div>
                                            </div>
                                            <div className="flex flex-col items-end gap-1 shrink-0">
                                                <span className="text-[10px] font-bold text-indigo-500">Seleccionar →</span>
                                                <button onClick={(e) => { e.stopPropagation(); toggleDetail(lens.id); }} className="text-[9px] text-[#312e81] underline hover:text-indigo-900 mt-1">
                                                    {isDetailOpen ? 'Menos info' : 'Ver detalle'}
                                                </button>
                                            </div>
                                        </div>
                                        {isDetailOpen && (
                                            <div className="p-2.5 bg-indigo-50/40 border-t border-indigo-100/50">
                                                {renderBullets(lens)}
                                            </div>
                                        )}
                                    </div>
                                );
                            })}
                        </div>
                    </div>
                );
            };

            const renderLcColumn = (colTitle, items) => {
                if(items.length === 0) return null;
                return (
                    <div className="flex flex-col gap-2 min-w-[280px] max-w-[280px] flex-1">
                        <div className="bg-[#eef2f6] border border-gray-200 px-3 py-2.5 rounded-t-lg shadow-sm">
                            <h4 className="font-bold text-[#312e81] text-[11px] text-center uppercase tracking-wider">{colTitle} <span className="text-gray-500 font-normal">({items.length})</span></h4>
                        </div>
                        <div className="flex flex-col gap-2.5 h-[500px] overflow-y-auto pr-2 pb-4 custom-scrollbar">
                            {items.map(lens => {
                                const isDetailOpen = !!lcExpandedDetails[lens.id];
                                return (
                                    <div key={lens.id} className="border border-gray-200 rounded-lg bg-white shadow-sm flex flex-col shrink-0 overflow-hidden hover:border-indigo-300 transition-colors">
                                        <div onClick={() => { setSelectedLc(lens); setLcModalOpen(true); }} className="p-3 cursor-pointer hover:bg-indigo-50/50 flex flex-col border-b border-gray-100">
                                            <div className="flex items-center gap-1.5 flex-wrap mb-2">
                                                {lens.badges && lens.badges.map((b, idx) => <span key={idx} className={getBadgeCls(b)}>{b}</span>)}
                                                {lens.promo && <span className={getPromoCls(lens.promo)}>{lens.promo}</span>}
                                            </div>
                                            <span className="text-[12px] font-bold text-gray-900 leading-snug mb-1">{lens.nombre}</span>
                                            <div className="text-[10px] text-gray-500 font-medium">{lens.tipo} · {lens.reemplazo}</div>
                                            <div className="flex justify-between items-center mt-3 pt-2.5 border-t border-gray-100/70">
                                                <span className="font-black text-gray-800 text-[12px]">{lens.precio} €</span>
                                                <span className="text-[10px] font-bold text-indigo-600 bg-indigo-50 px-2 py-0.5 rounded">Configurar →</span>
                                            </div>
                                        </div>
                                        <div className="bg-gray-50/50 px-3 py-1.5 flex justify-end">
                                            <button onClick={() => toggleDetail(lens.id, true)} className="text-[10px] text-[#312e81] underline font-medium hover:text-indigo-900">
                                                {isDetailOpen ? 'Ocultar info' : 'Ver detalle'}
                                            </button>
                                        </div>
                                        {isDetailOpen && (
                                            <div className="p-3.5 bg-indigo-50/60 border-t border-indigo-100">
                                                {renderBullets(lens)}
                                            </div>
                                        )}
                                    </div>
                                );
                            })}
                        </div>
                    </div>
                );
            };

            return (
                <div className="flex min-h-screen w-full font-sans text-gray-800">
                    <aside className={`bg-white border-r border-gray-200 flex flex-col hidden md:flex shrink-0 z-20 sticky top-0 h-screen transition-all duration-300 ${isSidebarCollapsed ? 'w-[75px]' : 'w-[245px]'}`}>
                        <div className="h-14 bg-cione-sidebar-brand flex items-center px-4 relative cursor-pointer" onClick={() => setIsSidebarCollapsed(!isSidebarCollapsed)}>
                            <div className="w-8 h-8 rounded-full bg-gradient-to-tr from-cyan-400 via-blue-500 to-fuchsia-500 mr-2.5 border-2 border-white/25 shadow-sm shrink-0"></div>
                            {!isSidebarCollapsed && <span className="font-bold text-white text-lg tracking-tight whitespace-nowrap">MyCione <span className="font-normal opacity-85">online</span></span>}
                        </div>
                        <nav className="flex-1 overflow-y-auto py-4 text-[13px] text-gray-600 overflow-x-hidden">
                            <div className="px-3 mb-3">
                                <button onClick={() => { setActiveCategory('inicio'); setShowCart(false); }} className={`w-full py-2 ${activeCategory === 'inicio' && !showCart ? 'bg-indigo-50/80 text-indigo-800' : 'text-gray-600 hover:bg-gray-50'} font-semibold rounded-md flex items-center transition-colors ${isSidebarCollapsed ? 'justify-center px-0' : 'px-3.5 gap-2.5 text-left'}`}>
                                    <svg className={`w-5 h-5 ${activeCategory === 'inicio' && !showCart ? 'text-indigo-700' : 'text-gray-400'} shrink-0`} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"></path></svg>
                                    {!isSidebarCollapsed && <span>Inicio</span>}
                                </button>
                            </div>
                            {!isSidebarCollapsed && <div className="mt-4 mb-1.5 px-5 text-[10px] font-bold text-gray-400 uppercase tracking-widest whitespace-nowrap">Óptica</div>}
                            <button onClick={() => { setActiveCategory('monturas'); setShowCart(false); }} className={`w-full flex items-center py-3 hover:bg-gray-50 transition-colors ${activeCategory === 'monturas' && !showCart ? 'font-bold text-gray-900 bg-gray-50 border-r-4 border-[#312e81]' : 'text-gray-600 border-r-4 border-transparent'} ${isSidebarCollapsed ? 'justify-center px-0' : 'justify-between px-5'}`}>
                                <span className={`flex items-center ${isSidebarCollapsed ? '' : 'gap-2.5'}`} title="Montura y gafas de sol">
                                    <svg className={`w-5 h-5 ${activeCategory === 'monturas' && !showCart ? 'text-[#312e81]' : 'text-gray-400'} shrink-0`} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.6" d="M4 12a3 3 0 106 0 3 3 0 00-6 0zm10 0a3 3 0 106 0 3 3 0 00-6 0zm-4 0h4"></path></svg>
                                    {!isSidebarCollapsed && <span className="whitespace-nowrap">Montura y gafas</span>}
                                </span>
                            </button>
                            <button onClick={() => { setActiveCategory('lentes_contacto'); setShowCart(false); }} className={`w-full flex items-center py-3 transition-colors ${activeCategory === 'lentes_contacto' && !showCart ? 'font-bold text-gray-900 bg-indigo-50/40 border-r-4 border-[#312e81]' : 'text-gray-600 hover:bg-gray-50 border-r-4 border-transparent'} ${isSidebarCollapsed ? 'justify-center px-0' : 'justify-between px-5'}`}>
                                <span className={`flex items-center ${isSidebarCollapsed ? '' : 'gap-2.5'}`} title="Lentes de contacto">
                                    <svg className={`w-5 h-5 ${activeCategory === 'lentes_contacto' && !showCart ? 'text-[#312e81]' : 'text-gray-400'} shrink-0`} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.6" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.6" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"></path></svg>
                                    {!isSidebarCollapsed && <span className="whitespace-nowrap">Lentes contacto</span>}
                                </span>
                            </button>
                            <button onClick={() => { setActiveCategory('lentes_graduadas'); setShowCart(false); }} className={`w-full flex items-center py-3 transition-colors ${activeCategory === 'lentes_graduadas' && !showCart ? 'font-bold text-gray-900 bg-indigo-50/40 border-r-4 border-[#312e81]' : 'text-gray-600 hover:bg-gray-50 border-r-4 border-transparent'} ${isSidebarCollapsed ? 'justify-center px-0' : 'justify-between px-5'}`}>
                                <span className={`flex items-center ${isSidebarCollapsed ? '' : 'gap-2.5'}`} title="Lentes graduadas">
                                    <svg className={`w-5 h-5 ${activeCategory === 'lentes_graduadas' && !showCart ? 'text-[#312e81]' : 'text-gray-400'} shrink-0`} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 6V4m0 2a2 2 0 100 4m0-4a2 2 0 110 4m-6 8a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4m6 6v10m6-2a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4"></path></svg>
                                    {!isSidebarCollapsed && <span className="whitespace-nowrap">Lentes graduadas</span>}
                                </span>
                            </button>
                            <button onClick={() => { setActiveCategory('audiologia'); setShowCart(false); }} className={`w-full flex items-center py-3 hover:bg-gray-50 transition-colors ${activeCategory === 'audiologia' && !showCart ? 'font-bold text-gray-900 bg-gray-50 border-r-4 border-[#312e81]' : 'text-gray-600 border-r-4 border-transparent'} ${isSidebarCollapsed ? 'justify-center px-0' : 'justify-between px-5'}`}>
                                <span className={`flex items-center ${isSidebarCollapsed ? '' : 'gap-2.5'}`} title="Audiología">
                                    <svg className={`w-5 h-5 ${activeCategory === 'audiologia' && !showCart ? 'text-[#312e81]' : 'text-gray-400'} shrink-0`} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.6" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z"></path></svg>
                                    {!isSidebarCollapsed && <span className="whitespace-nowrap">Audiología</span>}
                                </span>
                            </button>
                        </nav>
                        <div className="p-3 border-t border-gray-200">
                             <button onClick={() => setIsSidebarCollapsed(!isSidebarCollapsed)} className="w-full py-2 flex justify-center text-gray-400 hover:bg-gray-100 rounded" title={isSidebarCollapsed ? 'Expandir menú' : 'Colapsar menú'}>
                                <svg className={`w-5 h-5 transform transition-transform ${isSidebarCollapsed ? 'rotate-180' : ''}`} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7"></path></svg>
                             </button>
                        </div>
                    </aside>

                    <main className="flex-1 flex flex-col min-w-0 bg-[#f8f9fc]">
                        <header className="h-14 bg-cione-header text-white flex items-center justify-end px-6 shrink-0 z-10 shadow-sm">
                            <div className="flex items-center gap-5 text-xs font-medium text-white/90">
                                <a href="#" onClick={e => e.preventDefault()} className="hover:text-white transition-colors">€Ciones y beneficios</a>
                                <a href="#" onClick={e => e.preventDefault()} className="hover:text-white transition-colors">Customers</a>
                                <a href="#" onClick={e => e.preventDefault()} className="hover:text-white transition-colors hidden lg:inline">Contactos y ayuda</a>
                                <a href="#" onClick={e => e.preventDefault()} className="hover:text-white transition-colors font-semibold">Perfil</a>
                            </div>
                        </header>

                        <div className="bg-white border-b border-gray-200 px-6 py-2.5 flex flex-wrap items-center justify-between gap-4 sticky top-0 z-10">
                            <div className="flex items-center gap-3 flex-1 min-w-[260px]">
                                <button onClick={() => { setActiveCategory('lentes_graduadas'); setShowCart(false); }} className="w-8 h-8 rounded-md bg-[#2e2a85] text-white flex items-center justify-center hover:bg-indigo-900 transition-colors shrink-0">
                                    <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M15 19l-7-7 7-7"></path></svg>
                                </button>
                                <div className="relative w-full max-w-md">
                                    <svg className="w-3.5 h-3.5 absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
                                    <input type="text" placeholder="Buscar por referencia, marca, modelo o categoría..." className="w-full bg-gray-50/70 border border-gray-200/80 rounded-md py-1.5 pl-8 pr-3 text-xs text-gray-700 outline-none focus:bg-white focus:border-indigo-300" />
                                </div>
                            </div>
                            <div className="flex items-center gap-5 text-xs font-semibold text-gray-700">
                                <a href="#" onClick={e => e.preventDefault()} className="hover:text-[#312e81] flex items-center gap-1.5"><svg className="w-4 h-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>Presupuestos</a>
                                <a href="#" onClick={e => e.preventDefault()} className="hover:text-[#312e81] flex items-center gap-1.5"><svg className="w-4 h-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4"></path></svg>Pedidos</a>
                                <a href="#" onClick={(e) => { e.preventDefault(); setShowCart(true); }} className="hover:text-[#312e81] flex items-center gap-1.5 relative text-[#312e81]"><svg className="w-4 h-4 text-[#312e81]" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="1.8" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"></path></svg>Carrito{cartItems.length > 0 && (<span className="ml-1 px-1.5 py-0.5 bg-[#312e81] text-white text-[10px] font-bold rounded-full leading-none">{cartItems.length}</span>)}</a>
                            </div>
                        </div>

                        <div className="p-6 lg:p-8 flex-1 flex flex-col max-w-[1400px] w-full mx-auto animate-fade-in">
                            
                            {activeCategory === 'inicio' ? (
                                <div className="animate-fade-in space-y-6 pb-10">
                                    <div className="bg-gradient-to-b from-[#8aa1e8] to-[#4665d1] rounded-xl p-6 text-white shadow-sm relative overflow-hidden">
                                        <div className="flex flex-wrap justify-between items-start gap-4 relative z-10">
                                            <div>
                                                <h2 className="text-2xl font-bold mb-1 tracking-tight">Buenos días, Óptica Luna</h2>
                                                <p className="text-xs opacity-90">Aquí está el resumen de tu actividad y las novedades que pueden interesarte</p>
                                            </div>
                                            <div className="flex flex-wrap gap-3">
                                                <div className="flex items-center gap-2 bg-white/20 backdrop-blur-sm rounded-full pl-1.5 pr-3 py-1 text-[10px] font-bold">
                                                    <div className="w-6 h-6 rounded-full bg-indigo-900 text-white flex items-center justify-center font-bold text-[10px]">MR</div>
                                                    <div className="leading-tight">María Rubio<br/><span className="font-normal opacity-80">SAS</span></div>
                                                    <div className="w-5 h-5 rounded-full bg-[#312e81] flex items-center justify-center ml-1">✉</div>
                                                </div>
                                                <div className="flex items-center gap-2 bg-white/20 backdrop-blur-sm rounded-full pl-1.5 pr-3 py-1 text-[10px] font-bold">
                                                    <div className="w-6 h-6 rounded-full bg-indigo-900 text-white flex items-center justify-center font-bold text-[10px]">AL</div>
                                                    <div className="leading-tight">Alberto López<br/><span className="font-normal opacity-80">Comercial</span></div>
                                                    <div className="w-5 h-5 rounded-full bg-[#312e81] flex items-center justify-center ml-1">📞</div>
                                                </div>
                                            </div>
                                        </div>
                                        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-3 mt-6 relative z-10">
                                            {[
                                                { title: 'Pedidos pendientes', val: '3', sub: 'pedidos en curso', icon: '📦', text: 'text-indigo-600' },
                                                { title: 'Avisos importantes', val: '1', sub: 'incidencia requiere revisión', icon: '⚠️', text: 'text-rose-500' },
                                                { title: 'Documentos disponibles', val: '2', sub: 'nuevas facturas', icon: '📄', text: 'text-blue-600' },
                                                { title: 'Saldo €Cione', val: '1.234€', sub: 'disponibles', icon: '💰', text: 'text-emerald-600' }
                                            ].map((kpi, i) => (
                                                <div key={i} className="bg-white rounded-lg p-3.5 text-gray-800 shadow-sm relative hover:-translate-y-0.5 transition-transform cursor-pointer group">
                                                    <div className={`text-lg mb-2 ${kpi.text}`}>{kpi.icon}</div>
                                                    <div className="text-[9px] text-gray-500 font-bold uppercase tracking-wider mb-0.5">{kpi.title}</div>
                                                    <div className="flex items-baseline gap-1.5"><span className="text-2xl font-black leading-none">{kpi.val}</span> <span className="text-[10px] text-gray-400 font-medium">{kpi.sub}</span></div>
                                                    <div className="absolute top-3 right-3 text-gray-300 group-hover:text-gray-500">↗</div>
                                                </div>
                                            ))}
                                            <div className="bg-[#243c94] rounded-lg p-3.5 text-white shadow-sm relative overflow-hidden">
                                                <div className="text-[9px] text-white/70 font-bold uppercase tracking-wider mb-0.5">Consumo total</div>
                                                <div className="text-2xl font-black leading-none mb-1.5">16.234€</div>
                                                <div className="text-[9px] text-white/60 leading-tight pr-12">Próximo hito de fidelización:<br/>Te faltan 280€ para nivel Gold</div>
                                                <div className="absolute right-3 top-1/2 -translate-y-1/2 w-12 h-12 rounded-full border-[3px] border-blue-400/30 flex items-center justify-center bg-[#2a45a3]">
                                                    <div className="text-[10px] font-bold z-10">280€</div>
                                                    <svg className="absolute inset-0 w-full h-full -rotate-90" viewBox="0 0 36 36"><path className="text-blue-400" strokeDasharray="60, 100" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" fill="none" stroke="currentColor" strokeWidth="3" /></svg>
                                                </div>
                                                <div className="absolute top-2 right-2 text-white/30">↗</div>
                                            </div>
                                        </div>
                                    </div>

                                    {/* Accesos rápidos al flujo de compra unificado */}
                                    <div className="bg-white border border-indigo-100 rounded-xl p-4 shadow-sm flex flex-wrap items-center justify-between gap-4">
                                        <div>
                                            <span className="text-[10px] font-bold uppercase tracking-wider text-indigo-700 bg-indigo-50 px-2 py-0.5 rounded">Asistente Unificado de Compra</span>
                                            <h3 className="text-sm font-bold text-gray-900 mt-1">¿Qué necesitas pedir hoy para tu óptica?</h3>
                                        </div>
                                        <div className="flex flex-wrap gap-2">
                                            <button onClick={() => setActiveCategory('lentes_graduadas')} className="px-4 py-2 bg-[#312e81] text-white text-xs font-bold rounded-md hover:bg-indigo-900 transition shadow-sm">Lentes graduadas →</button>
                                            <button onClick={() => setActiveCategory('lentes_contacto')} className="px-4 py-2 bg-indigo-50 text-[#312e81] border border-indigo-200 text-xs font-bold rounded-md hover:bg-indigo-100 transition">Lentes de contacto →</button>
                                            <button onClick={() => setActiveCategory('monturas')} className="px-4 py-2 bg-white text-gray-700 border border-gray-200 text-xs font-bold rounded-md hover:bg-gray-50 transition">Monturas y sol</button>
                                            <button onClick={() => setActiveCategory('audiologia')} className="px-4 py-2 bg-white text-gray-700 border border-gray-200 text-xs font-bold rounded-md hover:bg-gray-50 transition">Audiología</button>
                                        </div>
                                    </div>

                                    <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                                        <div>
                                            <div className="flex justify-between items-center mb-3">
                                                <h3 className="font-bold text-gray-900 text-sm">Impulsa tu negocio</h3>
                                                <button className="text-[10px] font-bold text-gray-500 border border-gray-200 px-2 py-1 rounded hover:bg-gray-50">Ir a negocio y marketing</button>
                                            </div>
                                            <div className="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden flex cursor-pointer group hover:border-indigo-200 transition-colors h-36">
                                                <div className="w-1/3 bg-gradient-to-br from-amber-100 to-orange-200 flex items-center justify-center text-3xl">🕶️</div>
                                                <div className="w-2/3 p-4 flex flex-col justify-center relative">
                                                    <span className="text-[9px] font-bold text-pink-600 uppercase tracking-widest mb-1">Marketing</span>
                                                    <h4 className="font-bold text-gray-900 text-sm mb-1.5 leading-snug">Implementa la nueva campaña de verano 2026</h4>
                                                    <p className="text-[10px] text-gray-500 line-clamp-2">Materiales escaparate, PLV y creatividades digitales listas para activar en tu punto de venta.</p>
                                                    <div className="text-[10px] font-bold text-[#312e81] mt-2 group-hover:underline">Ver recursos ↗</div>
                                                </div>
                                            </div>
                                        </div>
                                        <div>
                                            <div className="flex justify-between items-center mb-3">
                                                <h3 className="font-bold text-gray-900 text-sm">Servicios de salud</h3>
                                                <button className="text-[10px] font-bold text-gray-500 border border-gray-200 px-2 py-1 rounded hover:bg-gray-50">Ir a salud</button>
                                            </div>
                                            <div className="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden flex cursor-pointer group hover:border-indigo-200 transition-colors h-36">
                                                <div className="w-1/3 bg-gradient-to-br from-teal-100 to-indigo-100 flex items-center justify-center text-3xl">👁️</div>
                                                <div className="w-2/3 p-4 flex flex-col justify-center relative">
                                                    <span className="text-[9px] font-bold text-teal-600 uppercase tracking-widest mb-1">Salud visual</span>
                                                    <h4 className="font-bold text-gray-900 text-sm mb-1.5 leading-snug">Herramientas de IA para mejorar las terapias visuales</h4>
                                                    <p className="text-[10px] text-gray-500 line-clamp-2">Protocolos clínicos digitales y seguimiento personalizado para tus pacientes en gabinete.</p>
                                                    <div className="text-[10px] font-bold text-[#312e81] mt-2 group-hover:underline">Descúbrelo ↗</div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>

                                    <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
                                        <div>
                                            <div className="flex justify-between items-center mb-4">
                                                <h3 className="font-bold text-gray-900 text-sm">Próximas formaciones</h3>
                                                <button className="text-[10px] font-bold text-gray-500 border border-gray-200 px-2 py-1 rounded hover:bg-gray-50">Ver todas</button>
                                            </div>
                                            <div className="grid grid-cols-2 gap-3">
                                                {[
                                                    { brand: 'HOYA', title: 'Ortoqueratología para todos', desc: 'Bloque I de iniciación. ¿Orto-K solo para miopes?' },
                                                    { brand: "mark'ennovy", title: 'Adaptación tórica personalizada', desc: 'Parámetros avanzados en geometrías blandas a medida.' },
                                                    { brand: 'TIEDRA', title: 'Control de miopía en gabinete', desc: 'Seguimiento de longitud axial y protocolos clínicos.' },
                                                    { brand: 'Fórum Contactología', title: 'Multifocales sin barreras', desc: 'Claves para el éxito en la primera adaptación présbita.' }
                                                ].map((item, i) => (
                                                    <div key={i} className="bg-white border border-gray-100 rounded-lg p-3.5 shadow-sm hover:border-indigo-200 cursor-pointer flex flex-col justify-between">
                                                        <div>
                                                            <div className="font-black text-indigo-900 text-xs tracking-wider mb-2">{item.brand}</div>
                                                            <div className="font-bold text-gray-900 text-[11px] leading-tight mb-1">{item.title}</div>
                                                            <div className="text-[9px] text-gray-500 leading-tight mb-4">{item.desc}</div>
                                                        </div>
                                                        <div className="flex justify-between items-end border-t border-gray-50 pt-2">
                                                            <div className="text-[9px] text-gray-400">Martes 9 JUN 10:00 h</div>
                                                            <div className="text-[9px] font-bold text-[#312e81]">Acceder al webinar</div>
                                                        </div>
                                                    </div>
                                                ))}
                                            </div>
                                        </div>
                                        <div>
                                            <div className="flex justify-between items-center mb-4">
                                                <h3 className="font-bold text-gray-900 text-sm">Temas que pueden interesarte</h3>
                                                <button className="text-[10px] font-bold text-gray-500 border border-gray-200 px-2 py-1 rounded hover:bg-gray-50">Ver todos</button>
                                            </div>
                                            <div className="space-y-2.5">
                                                {[
                                                    { icon: 'bd', color: 'text-purple-600 bg-purple-100', title: 'Miopía', sub: 'Soluciones ópticas y lentes de contacto de control' },
                                                    { icon: '👁', color: 'text-amber-600 bg-amber-100', title: 'Superficie ocular y ojo seco', sub: 'Protocolos de lágrima artificial y mantenimiento' },
                                                    { icon: 'IA', color: 'text-teal-600 bg-teal-100', title: 'Terapia visual e IA', sub: 'Nuevas herramientas diagnósticas para tu óptica' },
                                                    { icon: '👂', color: 'text-pink-600 bg-pink-100', title: 'Salud auditiva', sub: 'Integración de gabinete audiológico en óptica' },
                                                    { icon: '〰', color: 'text-blue-600 bg-blue-100', title: 'Audiometría con IA', sub: 'Optimización de ajustes en audífonos digitales' }
                                                ].map((t, i) => (
                                                    <div key={i} className="bg-white border border-gray-100 rounded-lg p-3 shadow-sm flex items-center justify-between hover:border-indigo-200 cursor-pointer group">
                                                        <div className="flex items-center gap-3">
                                                            <div className={`w-8 h-8 rounded text-xs font-black flex items-center justify-center ${t.color}`}>{t.icon}</div>
                                                            <div>
                                                                <div className="text-[11px] font-bold text-gray-900 leading-tight">{t.title}</div>
                                                                <div className="text-[9px] text-gray-500">{t.sub}</div>
                                                            </div>
                                                        </div>
                                                        <div className="text-gray-300 group-hover:text-indigo-400 mr-2 font-bold">↗</div>
                                                    </div>
                                                ))}
                                            </div>
                                        </div>
                                    </div>

                                    <div>
                                        <div className="flex justify-between items-center mb-4 mt-4">
                                            <h3 className="font-bold text-gray-900 text-sm">Actualidad Cione</h3>
                                            <button className="text-[10px] font-bold text-gray-500 border border-gray-200 px-2 py-1 rounded hover:bg-gray-50">Ver todas</button>
                                        </div>
                                        <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
                                            {[
                                                { bg: 'from-blue-100 to-indigo-100', emoji: '💧', date: 'Laura Fernández • 20 Jun 2026', title: 'Ojo seco en OPTOM 2026', tags: ['Optometría', 'Ojo seco'] },
                                                { bg: 'from-purple-100 to-pink-100', emoji: '🏛️', date: 'Sofía Guzmán • 12 Mar 2026', title: 'EXPOÓPTICA 2026', tags: ['Optometría', 'Miopía', 'Eventos'] },
                                                { bg: 'from-amber-100 to-yellow-100', emoji: '☀️', date: 'Isabela Ríos • 22 Jun 2026', title: 'Metamorfosis', tags: ['Verano', 'Colección'] },
                                                { bg: 'from-emerald-100 to-teal-100', emoji: '🚚', date: 'Amelia Stone • 12 Dic 2025', title: 'Nuevo compromiso logístico', tags: ['Logística'] }
                                            ].map((n, i) => (
                                                <div key={i} className="bg-white rounded-lg border border-gray-100 shadow-sm overflow-hidden hover:shadow flex flex-col group cursor-pointer">
                                                    <div className={`h-24 bg-gradient-to-br ${n.bg} flex items-center justify-center text-3xl group-hover:scale-105 transition-transform duration-300`}>
                                                        {n.emoji}
                                                    </div>
                                                    <div className="p-3 flex-1 flex flex-col">
                                                        <div className="text-[8px] text-gray-400 mb-1">{n.date}</div>
                                                        <h4 className="font-bold text-[11px] text-gray-900 mb-1.5 line-clamp-1">{n.title}</h4>
                                                        <p className="text-[9px] text-gray-500 line-clamp-2 mb-2">Novedades y recursos exclusivos para los socios del grupo Cione...</p>
                                                        <div className="mt-auto flex flex-wrap gap-1">
                                                            {n.tags.map((tg, idx) => <span key={idx} className="bg-gray-50 border border-gray-200 text-gray-600 px-1.5 py-0.5 rounded text-[8px] font-semibold">{tg}</span>)}
                                                        </div>
                                                    </div>
                                                </div>
                                            ))}
                                        </div>
                                    </div>
                                </div>
                            ) : activeCategory === 'lentes_graduadas' && !showCart ? (
                                <div>
                                    <div className="mb-5"><h1 className="text-2xl font-bold text-gray-900 tracking-tight">Asistente de Lentes Graduadas</h1></div>
                                    <div className="space-y-6">
                                        <div className="bg-white border border-gray-200 rounded-lg px-5 py-4 shadow-sm">
                                            
                                            <div className="mb-4">
                                                <span className="text-xs font-bold text-gray-800 block mb-2">Configuración del encargo</span>
                                                <div className="inline-flex bg-indigo-50/70 p-1 rounded-lg border border-indigo-100">
                                                    <button onClick={() => setOjoMode('ambos')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'ambos' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Ambos ojos</button>
                                                    <button onClick={() => setOjoMode('od')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'od' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Solo OD</button>
                                                    <button onClick={() => setOjoMode('oi')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'oi' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Solo OI</button>
                                                </div>
                                            </div>

                                            <h4 className="text-sm font-bold text-gray-900 flex items-center gap-1.5 mb-4 mt-6">
                                                Introduce la graduación de gafa
                                            </h4>
                                            
                                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6 w-full">
                                                {(ojoMode === 'ambos' || ojoMode === 'od') && (
                                                    <div className={`bg-gray-50/70 border border-gray-200/80 rounded-md px-4 py-3 relative transition-colors ${ojoMode === 'od' ? 'ring-1 ring-[#312e81]/30 border-[#312e81]' : ''}`}>
                                                        {ojoMode === 'ambos' && (
                                                            <button type="button" onClick={() => copyToLeft('grad')} className="absolute -top-3 right-2 bg-indigo-100 border border-indigo-200 text-[#312e81] text-[9px] font-bold px-2 py-0.5 rounded shadow-sm hover:bg-indigo-200 flex items-center gap-1 z-10 transition-colors">
                                                                Copiar datos a izq (OI) →
                                                            </button>
                                                        )}
                                                        <div className="flex items-center justify-between mb-2">
                                                            <h5 className="text-[12px] font-bold text-[#312e81]">Ojo derecho (OD)</h5>
                                                        </div>
                                                        <div className="grid grid-cols-4 gap-2">
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Esfera</label><input type="text" placeholder="-" value={asistenteGrad.odEsf} onChange={e => setAsistenteGrad({...asistenteGrad, odEsf: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-indigo-400 outline-none" /></div>
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Cilindro</label><input type="text" placeholder="-" value={asistenteGrad.odCil} onChange={e => setAsistenteGrad({...asistenteGrad, odCil: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-indigo-400 outline-none" /></div>
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Eje</label><input type="text" placeholder="-" value={asistenteGrad.odEje} onChange={e => setAsistenteGrad({...asistenteGrad, odEje: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-indigo-400 outline-none" /></div>
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Adición</label><input type="text" placeholder="-" value={asistenteGrad.odAdd} onChange={e => setAsistenteGrad({...asistenteGrad, odAdd: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold bg-indigo-50/40 focus:border-indigo-400 outline-none" /></div>
                                                        </div>
                                                    </div>
                                                )}

                                                {(ojoMode === 'ambos' || ojoMode === 'oi') && (
                                                    <div className={`bg-gray-50/70 border border-gray-200/80 rounded-md px-4 py-3 relative transition-colors ${ojoMode === 'oi' ? 'ring-1 ring-[#312e81]/30 border-[#312e81]' : ''}`}>
                                                        {ojoMode === 'ambos' && (
                                                            <button type="button" onClick={() => copyToRight('grad')} className="absolute -top-3 right-2 bg-indigo-100 border border-indigo-200 text-[#312e81] text-[9px] font-bold px-2 py-0.5 rounded shadow-sm hover:bg-indigo-200 flex items-center gap-1 z-10 transition-colors">
                                                                ← Copiar datos a der (OD)
                                                            </button>
                                                        )}
                                                        <div className="flex items-center justify-between mb-2">
                                                            <h5 className="text-[12px] font-bold text-[#312e81]">Ojo izquierdo (OI)</h5>
                                                        </div>
                                                        <div className="grid grid-cols-4 gap-2">
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Esfera</label><input type="text" placeholder="-" value={asistenteGrad.oiEsf} onChange={e => setAsistenteGrad({...asistenteGrad, oiEsf: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-indigo-400 outline-none" /></div>
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Cilindro</label><input type="text" placeholder="-" value={asistenteGrad.oiCil} onChange={e => setAsistenteGrad({...asistenteGrad, oiCil: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-indigo-400 outline-none" /></div>
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Eje</label><input type="text" placeholder="-" value={asistenteGrad.oiEje} onChange={e => setAsistenteGrad({...asistenteGrad, oiEje: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-indigo-400 outline-none" /></div>
                                                            <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5 text-center">Adición</label><input type="text" placeholder="-" value={asistenteGrad.oiAdd} onChange={e => setAsistenteGrad({...asistenteGrad, oiAdd: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold bg-indigo-50/40 focus:border-indigo-400 outline-none" /></div>
                                                        </div>
                                                    </div>
                                                )}
                                            </div>
                                            
                                            <div className="bg-white border-t border-gray-200 mt-6 pt-4 flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
                                                <div className="flex flex-wrap items-center gap-4 flex-1">
                                                    <div className="min-w-[140px]">
                                                        <h3 className="text-xs font-bold text-gray-900">Filtrar por proveedor</h3>
                                                        <div className="text-[10px] text-gray-500">{asistenteProv.length} proveedores seleccionados</div>
                                                    </div>
                                                    <div className="flex flex-wrap gap-1.5 flex-1">
                                                        {['Essilor', 'Hoya', 'Zeiss', 'Indo', 'Rodenstock', 'Otros'].map(p => {
                                                            const isSel = asistenteProv.includes(p);
                                                            return (
                                                                <button key={p} onClick={() => toggleAsistenteProv(p)} className={`px-3.5 py-1.5 rounded text-[11px] font-bold transition-colors border flex items-center justify-center min-w-[78px] ${isSel ? 'bg-[#312e81] text-white border-[#312e81]' : 'bg-gray-50 text-gray-600 border-gray-200 hover:bg-gray-100'}`}>
                                                                    {isSel && <span className="mr-1 font-black">✓</span>}{p}
                                                                </button>
                                                            );
                                                        })}
                                                    </div>
                                                </div>
                                                <div className="flex items-center gap-4 shrink-0">
                                                    {(asistenteProv.length > 0 || asistenteOnlyFavs || hasAdditionEntered) && (
                                                        <button onClick={() => { setAsistenteProv([]); setAsistenteOnlyFavs(false); setAsistenteGrad({ odEsf: '', odCil: '', odEje: '', odAdd: '', oiEsf: '', oiCil: '', oiEje: '', oiAdd: '' }); }} className="text-[11px] text-indigo-700 font-bold hover:underline">Limpiar filtros</button>
                                                    )}
                                                    <button onClick={() => setAsistenteOnlyFavs(!asistenteOnlyFavs)} className={`flex items-center gap-1.5 text-[11px] font-bold border px-3.5 py-1.5 rounded transition-colors ${asistenteOnlyFavs ? 'bg-amber-50 text-amber-800 border-amber-300' : 'text-gray-700 border-gray-200 hover:bg-gray-50 bg-white'}`}>
                                                        <span className="text-amber-500 text-xs">★</span> Favoritos
                                                    </button>
                                                </div>
                                            </div>
                                        </div>

                                        <div className="overflow-x-auto pb-4 custom-scrollbar">
                                            <h2 className="text-sm font-black text-gray-900 mb-3 uppercase tracking-wide">Producto propio Cione</h2>
                                            <div className="flex gap-4">
                                                {renderAsistenteGroup("Monofocales", propPropios.filter(l => l.familia === 'Monofocales'))}
                                                {renderAsistenteGroup("Progresivos", propPropios.filter(l => l.familia === 'Progresivos'))}
                                                {renderAsistenteGroup("Ocupacionales", propPropios.filter(l => l.familia === 'Ocupacionales'))}
                                                {renderAsistenteGroup("Bifocales", propPropios.filter(l => l.familia === 'Bifocales'))}
                                            </div>
                                        </div>

                                        <div className="pb-10 overflow-x-auto custom-scrollbar">
                                            <h2 className="text-sm font-black text-gray-900 mb-3 uppercase tracking-wide">Terceros</h2>
                                            <div className="flex gap-4">
                                                {renderAsistenteGroup("Monofocales", propTerceros.filter(l => l.familia === 'Monofocales'))}
                                                {renderAsistenteGroup("Progresivos", propTerceros.filter(l => l.familia === 'Progresivos'))}
                                                {renderAsistenteGroup("Ocupacionales", propTerceros.filter(l => l.familia === 'Ocupacionales'))}
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            ) : activeCategory === 'lentes_contacto' && !showCart ? (
                                /* ================= LENTES DE CONTACTO (ASISTENTE + BUSCAR POR FILTROS) ================= */
                                <div>
                                    <div className="mb-5"><h1 className="text-2xl font-bold text-gray-900 tracking-tight">Asistente de Lentes de Contacto</h1></div>
                                    <div className="space-y-6">
                                        
                                        <div className="bg-white border border-gray-200 rounded-lg px-5 py-4 shadow-sm">
                                            <div className="mb-4">
                                                <span className="text-xs font-bold text-gray-800 block mb-2">Configuración del encargo</span>
                                                <div className="inline-flex bg-indigo-50/70 p-1 rounded-lg border border-indigo-100">
                                                    <button onClick={() => setOjoMode('ambos')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'ambos' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Ambos ojos</button>
                                                    <button onClick={() => setOjoMode('od')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'od' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Solo OD</button>
                                                    <button onClick={() => setOjoMode('oi')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'oi' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Solo OI</button>
                                                </div>
                                            </div>

                                            <h4 className="text-sm font-bold text-gray-900 flex items-center gap-1.5 mb-4 mt-6">
                                                Graduación y parámetros
                                                <div className="relative group flex items-center">
                                                    <svg className="w-4 h-4 text-blue-500 cursor-help" fill="none" stroke="currentColor" viewBox="0 0 24 24"><circle cx="12" cy="12" r="10" strokeWidth="2"></circle><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 16v-4m0-4h.01M12 8h.01"></path></svg>
                                                    <div className="absolute left-1/2 -translate-x-1/2 bottom-full mb-2 hidden group-hover:block w-56 p-2 bg-gray-800 text-white text-[10px] rounded shadow-lg text-center z-50">
                                                        La graduación de lente de contacto se calcula de forma diferente a la de la gafa debido a la distancia al vértice. Recuerda aplicar la compensación para esferas mayores de ±4.00D.
                                                    </div>
                                                </div>
                                            </h4>
                                            
                                            <div className="grid grid-cols-1 md:grid-cols-2 gap-6 w-full">
                                                {(ojoMode === 'ambos' || ojoMode === 'od') && (
                                                    <div className={`bg-gray-50/70 border border-gray-200/80 rounded-md px-4 py-3 relative transition-colors ${ojoMode === 'od' ? 'ring-1 ring-[#312e81]/30 border-[#312e81]' : ''}`}>
                                                        {ojoMode === 'ambos' && (
                                                            <button type="button" onClick={() => copyToLeft('lc')} className="absolute -top-3 right-2 bg-indigo-100 border border-indigo-200 text-[#312e81] text-[9px] font-bold px-2 py-0.5 rounded shadow-sm hover:bg-indigo-200 flex items-center gap-1 z-10 transition-colors">
                                                                Copiar datos a izq (OI) →
                                                            </button>
                                                        )}
                                                        <div className="flex items-center justify-between mb-2">
                                                            <h5 className="text-[12px] font-bold text-[#312e81]">Ojo derecho (OD)</h5>
                                                        </div>
                                                        <div className="grid grid-cols-6 gap-2">
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Esfera</label><input type="text" placeholder="0.00" value={lcGrad.odEsf} onChange={e => setLcGrad({...lcGrad, odEsf: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold outline-none focus:border-indigo-400" /></div>
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Cilindro</label><input type="text" placeholder="-" value={lcGrad.odCil} onChange={e => setLcGrad({...lcGrad, odCil: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold outline-none focus:border-indigo-400" /></div>
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Eje</label><input type="text" placeholder="-" value={lcGrad.odEje} onChange={e => setLcGrad({...lcGrad, odEje: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold outline-none focus:border-indigo-400" /></div>
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Adición</label><input type="text" placeholder="-" value={lcGrad.odAdd} onChange={e => setLcGrad({...lcGrad, odAdd: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold bg-indigo-50/40 outline-none focus:border-indigo-400" /></div>
                                                            <div className="border-l border-gray-200 pl-2">
                                                                <label className="block text-[9px] font-bold text-[#312e81] mb-0.5 text-center">Radio (R)</label>
                                                                <input type="text" placeholder="8,6" value={lcGrad.odRadio} onChange={e => setLcGrad({...lcGrad, odRadio: e.target.value})} className="w-full border border-indigo-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold bg-white outline-none focus:border-indigo-500" />
                                                            </div>
                                                            <div>
                                                                <label className="block text-[9px] font-bold text-[#312e81] mb-0.5 text-center">Diám. (Ø)</label>
                                                                <input type="text" placeholder="14,2" value={lcGrad.odDiam} onChange={e => setLcGrad({...lcGrad, odDiam: e.target.value})} className="w-full border border-indigo-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold bg-white outline-none focus:border-indigo-500" />
                                                            </div>
                                                        </div>
                                                    </div>
                                                )}

                                                {(ojoMode === 'ambos' || ojoMode === 'oi') && (
                                                    <div className={`bg-gray-50/70 border border-gray-200/80 rounded-md px-4 py-3 relative transition-colors ${ojoMode === 'oi' ? 'ring-1 ring-[#312e81]/30 border-[#312e81]' : ''}`}>
                                                        {ojoMode === 'ambos' && (
                                                            <button type="button" onClick={() => copyToRight('lc')} className="absolute -top-3 right-2 bg-indigo-100 border border-indigo-200 text-[#312e81] text-[9px] font-bold px-2 py-0.5 rounded shadow-sm hover:bg-indigo-200 flex items-center gap-1 z-10 transition-colors">
                                                                ← Copiar datos a der (OD)
                                                            </button>
                                                        )}
                                                        <div className="flex items-center justify-between mb-2">
                                                            <h5 className="text-[12px] font-bold text-[#312e81]">Ojo izquierdo (OI)</h5>
                                                        </div>
                                                        <div className="grid grid-cols-6 gap-2">
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Esfera</label><input type="text" placeholder="0.00" value={lcGrad.oiEsf} onChange={e => setLcGrad({...lcGrad, oiEsf: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold outline-none focus:border-indigo-400" /></div>
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Cilindro</label><input type="text" placeholder="-" value={lcGrad.oiCil} onChange={e => setLcGrad({...lcGrad, oiCil: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold outline-none focus:border-indigo-400" /></div>
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Eje</label><input type="text" placeholder="-" value={lcGrad.oiEje} onChange={e => setLcGrad({...lcGrad, oiEje: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold outline-none focus:border-indigo-400" /></div>
                                                            <div><label className="block text-[9px] font-semibold text-gray-600 mb-0.5 text-center">Adición</label><input type="text" placeholder="-" value={lcGrad.oiAdd} onChange={e => setLcGrad({...lcGrad, oiAdd: e.target.value})} className="w-full border border-gray-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold bg-indigo-50/40 outline-none focus:border-indigo-400" /></div>
                                                            <div className="border-l border-gray-200 pl-2">
                                                                <label className="block text-[9px] font-bold text-[#312e81] mb-0.5 text-center">Radio (R)</label>
                                                                <input type="text" placeholder="8,6" value={lcGrad.oiRadio} onChange={e => setLcGrad({...lcGrad, oiRadio: e.target.value})} className="w-full border border-indigo-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold bg-white outline-none focus:border-indigo-500" />
                                                            </div>
                                                            <div>
                                                                <label className="block text-[9px] font-bold text-[#312e81] mb-0.5 text-center">Diám. (Ø)</label>
                                                                <input type="text" placeholder="14,2" value={lcGrad.oiDiam} onChange={e => setLcGrad({...lcGrad, oiDiam: e.target.value})} className="w-full border border-indigo-300 rounded px-1 py-1.5 text-[11px] text-center font-semibold bg-white outline-none focus:border-indigo-500" />
                                                            </div>
                                                        </div>
                                                    </div>
                                                )}
                                            </div>
                                        </div>

                                        <div className="pt-4">
                                            <div className="flex items-center gap-2 mb-3">
                                                <h2 className="text-sm font-black text-gray-900 uppercase tracking-wide">Producto propio Cione ({FULL_LC_CATALOG.filter(l => l.prop).length} resultados)</h2>
                                            </div>
                                            <div className="flex gap-4 overflow-x-auto pb-2 custom-scrollbar">
                                                {renderLcColumn('Cione Basic', FULL_LC_CATALOG.filter(l => l.prop && matchesLcItem(l) && l.cat_propio === 'Cione Basic'))}
                                                {renderLcColumn('Cione Premium', FULL_LC_CATALOG.filter(l => l.prop && matchesLcItem(l) && l.cat_propio === 'Cione Premium'))}
                                                {renderLcColumn('Cione Advanced', FULL_LC_CATALOG.filter(l => l.prop && matchesLcItem(l) && l.cat_propio === 'Cione Advanced'))}
                                            </div>
                                        </div>

                                        <div className="pt-4 pb-10">
                                            <div className="flex items-center gap-2 mb-3">
                                                <h2 className="text-sm font-black text-gray-900 uppercase tracking-wide">Terceros ({FULL_LC_CATALOG.filter(l => !l.prop).length} resultados)</h2>
                                            </div>
                                            <div className="flex gap-4 overflow-x-auto pb-2 custom-scrollbar">
                                                {renderLcColumn('Alcon', FULL_LC_CATALOG.filter(l => !l.prop && matchesLcItem(l) && l.fab === 'Alcon'))}
                                                {renderLcColumn('Bausch & Lomb', FULL_LC_CATALOG.filter(l => !l.prop && matchesLcItem(l) && l.fab === 'Bausch & Lomb'))}
                                                {renderLcColumn('CooperVision', FULL_LC_CATALOG.filter(l => !l.prop && matchesLcItem(l) && l.fab === 'CooperVision'))}
                                                {renderLcColumn('Johnson & Johnson', FULL_LC_CATALOG.filter(l => !l.prop && matchesLcItem(l) && l.fab === 'Johnson & Johnson'))}
                                                {renderLcColumn('Mark\'ennovy', FULL_LC_CATALOG.filter(l => !l.prop && matchesLcItem(l) && l.fab === 'Mark\'ennovy'))}
                                            </div>
                                        </div>

                                    </div>
                                </div>
                            ) : (
                                <div className="bg-white border border-gray-200 rounded-xl p-8 shadow-sm mb-8 animate-fade-in">
                                    <div className="flex items-center justify-between mb-6 pb-4 border-b border-gray-100">
                                        <div>
                                            <span className="text-[11px] font-bold uppercase tracking-wider text-indigo-700 bg-indigo-50 px-2.5 py-1 rounded">Sección en construcción</span>
                                            <h2 className="text-2xl font-bold text-gray-900 mt-2">{activeCategory === 'monturas' ? 'Asistente de Monturas y Gafas de Sol' : 'Asistente de Audiología'}</h2>
                                            <p className="text-sm text-gray-500 mt-1">Este módulo estará disponible en futuras versiones de la plataforma unificada.</p>
                                        </div>
                                    </div>
                                </div>
                            )}

                        </div>
                    </main>

                    {/* MODAL CONFIGURACIÓN LENTE GRADUADA COMPLETO */}
                    {modalOpen && (
                        <div className="fixed inset-0 bg-black/60 z-50 flex items-center justify-center p-4 lg:p-6 backdrop-blur-sm">
                            <div className="bg-white rounded-xl shadow-2xl w-full max-w-6xl max-h-[94vh] flex flex-col font-sans overflow-hidden">
                                <div className="bg-white border-b border-gray-200 px-6 py-4 flex justify-between items-center z-10 shrink-0">
                                    <div><h2 className="text-lg font-bold text-gray-900">Configurar lentes graduadas</h2><p className="text-xs text-gray-500 mt-0.5">Selecciona el producto y las características de cada ojo.</p></div>
                                    <button onClick={() => setModalOpen(false)} className="px-3.5 py-1.5 bg-white border border-gray-300 rounded text-xs font-semibold text-gray-700 hover:bg-gray-50 transition">Cerrar</button>
                                </div>
                                <div className="flex-1 overflow-y-auto p-6 bg-gray-50/40">
                                    <div className="mb-5">
                                        <span className="text-xs font-bold text-gray-800 block mb-2">Encargo</span>
                                        <div className="inline-flex bg-indigo-50/70 p-1 rounded-lg border border-indigo-100">
                                            <button onClick={() => setOjoMode('ambos')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'ambos' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Ambos ojos</button>
                                            <button onClick={() => setOjoMode('od')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'od' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Solo OD</button>
                                            <button onClick={() => setOjoMode('oi')} className={`px-4 py-1.5 text-xs font-bold rounded-md transition-all ${ojoMode === 'oi' ? 'bg-[#312e81] shadow-sm text-white' : 'text-gray-700 hover:bg-white/60'}`}>Solo OI</button>
                                        </div>
                                    </div>

                                    <div className={`grid ${ojoMode === 'ambos' ? 'grid-cols-1 lg:grid-cols-2' : 'grid-cols-1 max-w-2xl'} gap-6 mb-6`}>
                                        {(ojoMode === 'ambos' || ojoMode === 'od') && (
                                            <div className={`bg-white border p-5 rounded-lg shadow-sm transition-colors ${ojoMode === 'od' ? 'border-[#312e81] ring-1 ring-[#312e81]/30' : 'border-gray-200'}`}>
                                                <div className="flex justify-between items-center mb-3">
                                                    <h3 className="font-bold text-[#312e81] text-sm">Ojo derecho (OD)</h3>
                                                </div>
                                                {ojoMode === 'od' && (
                                                    <div className="mb-4 flex justify-end">
                                                        <button type="button" onClick={() => copyToLeft('grad')} className="text-[10px] bg-indigo-50 border border-indigo-100 text-[#312e81] font-bold px-3 py-1.5 rounded hover:bg-indigo-100 flex items-center gap-1.5 transition-colors shadow-sm">
                                                            <svg className="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M14 5l7 7m0 0l-7 7m7-7H3"></path></svg>
                                                            Copiar datos a izq (OI)
                                                        </button>
                                                    </div>
                                                )}
                                                <div className="mb-4 pb-3 border-b border-gray-100">
                                                    <span className="inline-block text-[10px] font-semibold text-gray-600 bg-gray-100 px-2 py-0.5 rounded mb-1">{selectedLens?.tag === 'PRODUCTO PROPIO CIONE' ? 'Producto propio Cione' : 'Terceros'}</span>
                                                    <div className="font-bold text-gray-900 text-sm flex flex-wrap items-center gap-2">
                                                        <span>{selectedLens?.producto}</span>
                                                        <button onClick={() => setModalOpen(false)} className="text-[10px] text-[#312e81] hover:underline font-bold bg-indigo-50/70 px-2 py-0.5 rounded border border-indigo-200 shadow-sm transition-colors hover:bg-indigo-100">Cambiar lente</button>
                                                    </div>
                                                    <div className="text-xs text-gray-500 mt-1">{selectedLens?.ref} · {selectedLens?.fab} · <span className="bg-blue-50 text-blue-700 px-1.5 py-0.5 rounded font-semibold">{selectedLens?.familia}</span></div>
                                                </div>
                                                <h4 className="text-xs font-bold text-gray-800 mb-2.5">Características de la lente</h4>
                                                <div className="grid grid-cols-1 sm:grid-cols-3 gap-2.5 mb-3">
                                                    <div><label className="block text-[11px] font-semibold text-gray-600 mb-1">Diseño</label><select value={modalSpecs.odDiseno} onChange={e=>setModalSpecs({...modalSpecs, odDiseno:e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs bg-white text-gray-800"><option value="">Seleccionar</option>{DISEÑOS.map((d,i) => <option key={i} value={d}>{d}</option>)}</select></div>
                                                    <div><label className="block text-[11px] font-semibold text-gray-600 mb-1">Índice</label><select value={modalSpecs.odIndice} onChange={e=>setModalSpecs({...modalSpecs, odIndice:e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs bg-white text-gray-800"><option value="">Seleccionar</option>{INDICES.map((ind, i) => <option key={i} value={ind}>{ind}</option>)}</select></div>
                                                    <div><label className="block text-[11px] font-semibold text-gray-600 mb-1">Tratamiento</label><select value={modalSpecs.odTratamiento} onChange={e=>setModalSpecs({...modalSpecs, odTratamiento:e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs bg-white text-gray-800"><option value="">Seleccionar</option>{TRATAMIENTOS.map((t,i) => <option key={i} value={t}>{t}</option>)}</select></div>
                                                </div>
                                                <h4 className="text-xs font-bold text-gray-800 mb-2.5">Graduación</h4>
                                                <div className="grid grid-cols-4 gap-2.5">
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Esfera</label><input type="text" placeholder="-" value={modalGrad.odEsf} onChange={e => setModalGrad({...modalGrad, odEsf: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold" /></div>
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Cilindro</label><input type="text" placeholder="-" value={modalGrad.odCil} onChange={e => setModalGrad({...modalGrad, odCil: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold" /></div>
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Eje</label><input type="text" placeholder="-" value={modalGrad.odEje} onChange={e => setModalGrad({...modalGrad, odEje: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold" /></div>
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Adición</label><input type="text" placeholder="-" value={modalGrad.odAdd} onChange={e => setModalGrad({...modalGrad, odAdd: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold bg-gray-50" /></div>
                                                </div>
                                            </div>
                                        )}
                                        {(ojoMode === 'ambos' || ojoMode === 'oi') && (
                                            <div className={`bg-white border p-5 rounded-lg shadow-sm transition-colors ${ojoMode === 'oi' ? 'border-[#312e81] ring-1 ring-[#312e81]/30' : 'border-gray-200'}`}>
                                                <div className="flex justify-between items-center mb-3">
                                                    <h3 className="font-bold text-[#312e81] text-sm">Ojo izquierdo (OI)</h3>
                                                </div>
                                                {ojoMode === 'oi' && (
                                                    <div className="mb-4 flex justify-end">
                                                        <button type="button" onClick={() => copyToRight('grad')} className="text-[10px] bg-indigo-50 border border-indigo-100 text-[#312e81] font-bold px-3 py-1.5 rounded hover:bg-indigo-100 flex items-center gap-1.5 transition-colors shadow-sm">
                                                            <svg className="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"></path></svg>
                                                            Copiar datos a der (OD)
                                                        </button>
                                                    </div>
                                                )}
                                                <div className="mb-4 pb-3 border-b border-gray-100">
                                                    <span className="inline-block text-[10px] font-semibold text-gray-600 bg-gray-100 px-2 py-0.5 rounded mb-1">{selectedLens?.tag === 'PRODUCTO PROPIO CIONE' ? 'Producto propio Cione' : 'Terceros'}</span>
                                                    <div className="font-bold text-gray-900 text-sm flex flex-wrap items-center gap-2">
                                                        <span>{selectedLens?.producto}</span>
                                                        <button onClick={() => setModalOpen(false)} className="text-[10px] text-[#312e81] hover:underline font-bold bg-indigo-50/70 px-2 py-0.5 rounded border border-indigo-200 shadow-sm transition-colors hover:bg-indigo-100">Cambiar lente</button>
                                                    </div>
                                                    <div className="text-xs text-gray-500 mt-1">{selectedLens?.ref} · {selectedLens?.fab} · <span className="bg-blue-50 text-blue-700 px-1.5 py-0.5 rounded font-semibold">{selectedLens?.familia}</span></div>
                                                </div>
                                                <h4 className="text-xs font-bold text-gray-800 mb-2.5">Características de la lente</h4>
                                                <div className="grid grid-cols-1 sm:grid-cols-3 gap-2.5 mb-3">
                                                    <div><label className="block text-[11px] font-semibold text-gray-600 mb-1">Diseño</label><select value={modalSpecs.oiDiseno} onChange={e=>setModalSpecs({...modalSpecs, oiDiseno:e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs bg-white text-gray-800"><option value="">Seleccionar</option>{DISEÑOS.map((d,i) => <option key={i} value={d}>{d}</option>)}</select></div>
                                                    <div><label className="block text-[11px] font-semibold text-gray-600 mb-1">Índice</label><select value={modalSpecs.oiIndice} onChange={e=>setModalSpecs({...modalSpecs, oiIndice:e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs bg-white text-gray-800"><option value="">Seleccionar</option>{INDICES.map((ind, i) => <option key={i} value={ind}>{ind}</option>)}</select></div>
                                                    <div><label className="block text-[11px] font-semibold text-gray-600 mb-1">Tratamiento</label><select value={modalSpecs.oiTratamiento} onChange={e=>setModalSpecs({...modalSpecs, oiTratamiento:e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs bg-white text-gray-800"><option value="">Seleccionar</option>{TRATAMIENTOS.map((t,i) => <option key={i} value={t}>{t}</option>)}</select></div>
                                                </div>
                                                <h4 className="text-xs font-bold text-gray-800 mb-2.5">Graduación</h4>
                                                <div className="grid grid-cols-4 gap-2.5">
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Esfera</label><input type="text" placeholder="-" value={modalGrad.oiEsf} onChange={e => setModalGrad({...modalGrad, oiEsf: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold" /></div>
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Cilindro</label><input type="text" placeholder="-" value={modalGrad.oiCil} onChange={e => setModalGrad({...modalGrad, oiCil: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold" /></div>
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Eje</label><input type="text" placeholder="-" value={modalGrad.oiEje} onChange={e => setModalGrad({...modalGrad, oiEje: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold" /></div>
                                                    <div><label className="block text-[11px] text-gray-600 font-semibold mb-1">Adición</label><input type="text" placeholder="-" value={modalGrad.oiAdd} onChange={e => setModalGrad({...modalGrad, oiAdd: e.target.value})} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold bg-gray-50" /></div>
                                                </div>
                                            </div>
                                        )}
                                    </div>

                                    <div className="bg-[#f8f9fc] border border-gray-200 rounded-lg p-5 shadow-sm">
                                        <h3 className="font-bold text-gray-900 text-sm mb-4">Resumen del encargo</h3>
                                        <div className="space-y-4">
                                            {(ojoMode === 'ambos' || ojoMode === 'od') && (
                                                <div className="pb-4 border-b border-gray-200">
                                                    <div className="flex items-center gap-2 mb-1.5">
                                                        <span className="font-bold text-gray-900 text-sm">OD · {selectedLens?.producto}</span>
                                                        <span className="bg-[#d1fae5] text-[#065f46] px-2 py-0.5 text-[10px] font-bold rounded">{selectedLens?.familia}</span>
                                                    </div>
                                                    <div className="text-xs text-gray-500 mb-0.5">{modalSpecs.odIndice || 'Índice base'} · {modalSpecs.odTratamiento || 'Tratamiento base'}</div>
                                                    <div className="text-xs text-gray-500 mb-2">Sin servicio adicional</div>
                                                    <div className="text-xs text-gray-700">Precio por lente · <span className="font-bold">{selectedLens?.precio} €</span> sin IVA</div>
                                                    <div className="text-xs text-gray-500">{selectedLens?.eciones || '0 EuroCiones'}</div>
                                                </div>
                                            )}
                                            {(ojoMode === 'ambos' || ojoMode === 'oi') && (
                                                <div className="pb-4 border-b border-gray-200">
                                                    <div className="flex items-center gap-2 mb-1.5">
                                                        <span className="font-bold text-gray-900 text-sm">OI · {selectedLens?.producto}</span>
                                                        <span className="bg-[#d1fae5] text-[#065f46] px-2 py-0.5 text-[10px] font-bold rounded">{selectedLens?.familia}</span>
                                                    </div>
                                                    <div className="text-xs text-gray-500 mb-0.5">{modalSpecs.oiIndice || 'Índice base'} · {modalSpecs.oiTratamiento || 'Tratamiento base'}</div>
                                                    <div className="text-xs text-gray-500 mb-2">Sin servicio adicional</div>
                                                    <div className="text-xs text-gray-700">Precio por lente · <span className="font-bold">{selectedLens?.precio} €</span> sin IVA</div>
                                                    <div className="text-xs text-gray-500">{selectedLens?.eciones || '0 EuroCiones'}</div>
                                                </div>
                                            )}
                                        </div>
                                        <div className="mt-4 flex flex-wrap justify-between items-center gap-4">
                                            <div className="font-bold text-gray-900 text-sm">
                                                Total · {formatPrice(modalTotalPrice)} € sin IVA
                                            </div>
                                        </div>
                                    </div>
                                </div>

                                <div className="bg-white border-t border-gray-200 px-6 py-4 flex flex-wrap justify-between items-center gap-4 shrink-0 z-10">
                                    <div><p className="text-xs font-bold text-gray-700">Precio trabajos de taller y lentes</p></div>
                                    <div className="flex items-center gap-5">
                                        <div className="text-right flex items-baseline gap-2 whitespace-nowrap">
                                            <span className="text-xs text-gray-400 line-through">{formatPrice(modalTotalPrice + 4.20 * activeEyesCount)} € PVP</span>
                                            <span className="text-xl font-black text-[#312e81]">{formatPrice(modalTotalPrice)} € <span className="text-xs font-bold text-gray-500">PVO</span></span>
                                        </div>
                                        <div className="flex gap-2.5">
                                            <button onClick={() => setModalOpen(false)} className="px-5 py-2 bg-white border border-gray-300 rounded-md text-xs font-bold text-gray-700 hover:bg-gray-50 transition">Cancelar</button>
                                            <button onClick={handleAddToCart} className="px-6 py-2 bg-[#312e81] rounded-md text-xs font-bold text-white hover:bg-indigo-900 shadow-sm transition">Añadir al carrito</button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    )}

                    {/* MODAL CONFIGURACIÓN LENTES CONTACTO */}
                    {lcModalOpen && selectedLc && (
                        <div className="fixed inset-0 bg-black/60 z-50 flex items-center justify-center p-4 backdrop-blur-sm">
                            <div className="bg-white rounded-xl shadow-2xl w-full max-w-4xl max-h-[92vh] flex flex-col overflow-hidden animate-fade-in">
                                <div className="bg-white border-b border-gray-200 px-6 py-4 flex justify-between items-center shrink-0">
                                    <div>
                                        <div className="flex items-center gap-2">
                                            <span className="text-[10px] font-bold uppercase tracking-wider text-indigo-700 bg-indigo-50 px-2 py-0.5 rounded">{selectedLc.prop ? 'Producto propio Cione' : 'Terceros'}</span>
                                            <span className="text-[10px] font-semibold text-gray-500">{selectedLc.reemplazo} · {selectedLc.material}</span>
                                        </div>
                                        <h3 className="text-base font-bold text-gray-900 mt-1">{selectedLc.nombre}</h3>
                                        <p className="text-xs text-gray-500">{selectedLc.ref} · {selectedLc.fab} · {selectedLc.tipo} · {selectedLc.formato}</p>
                                    </div>
                                    <button onClick={() => setLcModalOpen(false)} className="px-3.5 py-1.5 border border-gray-300 rounded text-xs font-semibold text-gray-700 hover:bg-gray-50">Cerrar</button>
                                </div>

                                <div className="p-6 bg-gray-50/40 overflow-y-auto space-y-4">
                                    {(!lcGrad.odEsf && !lcGrad.oiEsf && !lcGrad.odCil && !lcGrad.oiCil) && (
                                        <div className="bg-indigo-50/80 border border-indigo-200/80 rounded-lg px-4 py-2.5 flex flex-wrap items-center justify-between gap-2">
                                            <div className="text-xs text-indigo-950">
                                                <span className="font-bold">Graduación pendiente:</span> No has introducido graduación previa. Puedes configurarla directamente aquí antes de añadir al carrito.
                                            </div>
                                        </div>
                                    )}

                                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                                        <div className="bg-white border border-gray-200 rounded-lg p-4 shadow-sm">
                                            <div className="flex items-center justify-between mb-3">
                                                <div className="font-bold text-xs text-[#312e81]">Ojo derecho (OD)</div>
                                            </div>
                                            <div className="grid grid-cols-3 gap-2 mb-3">
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Esfera</label><input type="text" placeholder="0,00" value={lcGrad.odEsf} onChange={e => setLcGrad({ ...lcGrad, odEsf: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Cilindro</label><input type="text" placeholder="-" value={lcGrad.odCil} onChange={e => setLcGrad({ ...lcGrad, odCil: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Eje</label><input type="text" placeholder="-" value={lcGrad.odEje} onChange={e => setLcGrad({ ...lcGrad, odEje: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Adición</label><input type="text" placeholder="-" value={lcGrad.odAdd} onChange={e => setLcGrad({ ...lcGrad, odAdd: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold bg-indigo-50/30 focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-bold text-[#312e81] mb-0.5">Radio (R)</label><input type="text" placeholder="8,6" value={lcGrad.odRadio} onChange={e => setLcGrad({ ...lcGrad, odRadio: e.target.value })} className="w-full border border-indigo-200 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-bold text-[#312e81] mb-0.5">Diámetro (Ø)</label><input type="text" placeholder="14,2" value={lcGrad.odDiam} onChange={e => setLcGrad({ ...lcGrad, odDiam: e.target.value })} className="w-full border border-indigo-200 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                            </div>
                                            <div className="pt-2.5 border-t border-gray-100 flex items-center justify-between">
                                                <label className="text-[11px] font-bold text-gray-700">Cantidad de cajas (OD)</label>
                                                <div className="flex items-center gap-2">
                                                    <button type="button" onClick={() => setLcBoxes(p => ({ ...p, od: Math.max(0, p.od - 1) }))} className="w-7 h-7 border border-gray-300 rounded bg-gray-50 hover:bg-gray-100 font-bold">−</button>
                                                    <span className="w-8 text-center text-xs font-bold">{lcBoxes.od}</span>
                                                    <button type="button" onClick={() => setLcBoxes(p => ({ ...p, od: p.od + 1 }))} className="w-7 h-7 border border-gray-300 rounded bg-gray-50 hover:bg-gray-100 font-bold">+</button>
                                                </div>
                                            </div>
                                        </div>

                                        <div className="bg-white border border-gray-200 rounded-lg p-4 shadow-sm">
                                            <div className="flex items-center justify-between mb-3">
                                                <div className="font-bold text-xs text-[#312e81]">Ojo izquierdo (OI)</div>
                                            </div>
                                            <div className="grid grid-cols-3 gap-2 mb-3">
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Esfera</label><input type="text" placeholder="0,00" value={lcGrad.oiEsf} onChange={e => setLcGrad({ ...lcGrad, oiEsf: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Cilindro</label><input type="text" placeholder="-" value={lcGrad.oiCil} onChange={e => setLcGrad({ ...lcGrad, oiCil: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Eje</label><input type="text" placeholder="-" value={lcGrad.oiEje} onChange={e => setLcGrad({ ...lcGrad, oiEje: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-semibold text-gray-600 mb-0.5">Adición</label><input type="text" placeholder="-" value={lcGrad.oiAdd} onChange={e => setLcGrad({ ...lcGrad, oiAdd: e.target.value })} className="w-full border border-gray-300 rounded px-2 py-1.5 text-xs text-center font-semibold bg-indigo-50/30 focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-bold text-[#312e81] mb-0.5">Radio (R)</label><input type="text" placeholder="8,6" value={lcGrad.oiRadio} onChange={e => setLcGrad({ ...lcGrad, oiRadio: e.target.value })} className="w-full border border-indigo-200 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                                <div><label className="block text-[10px] font-bold text-[#312e81] mb-0.5">Diámetro (Ø)</label><input type="text" placeholder="14,2" value={lcGrad.oiDiam} onChange={e => setLcGrad({ ...lcGrad, oiDiam: e.target.value })} className="w-full border border-indigo-200 rounded px-2 py-1.5 text-xs text-center font-semibold focus:border-[#312e81] outline-none" /></div>
                                            </div>
                                            <div className="pt-2.5 border-t border-gray-100 flex items-center justify-between">
                                                <label className="text-[11px] font-bold text-gray-700">Cantidad de cajas (OI)</label>
                                                <div className="flex items-center gap-2">
                                                    <button type="button" onClick={() => setLcBoxes(p => ({ ...p, oi: Math.max(0, p.oi - 1) }))} className="w-7 h-7 border border-gray-300 rounded bg-gray-50 hover:bg-gray-100 font-bold">−</button>
                                                    <span className="w-8 text-center text-xs font-bold">{lcBoxes.oi}</span>
                                                    <button type="button" onClick={() => setLcBoxes(p => ({ ...p, oi: p.oi + 1 }))} className="w-7 h-7 border border-gray-300 rounded bg-gray-50 hover:bg-gray-100 font-bold">+</button>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>

                                <div className="bg-white border-t border-gray-200 px-6 py-4 flex flex-wrap justify-between items-center gap-4 shrink-0">
                                    <div>
                                        <span className="text-xs text-gray-500">Total sin IVA ({lcBoxes.od + lcBoxes.oi} cajas): </span>
                                        <span className="text-lg font-black text-[#312e81]">{formatPrice(parsePrice(selectedLc.precio) * (lcBoxes.od + lcBoxes.oi))} €</span>
                                    </div>
                                    <div className="flex gap-2">
                                        <button onClick={() => setLcModalOpen(false)} className="px-4 py-2 border border-gray-300 rounded-md text-xs font-bold text-gray-700 hover:bg-gray-50">Cancelar</button>
                                        <button onClick={handleAddLcToCart} className="px-5 py-2 bg-[#312e81] text-white rounded-md text-xs font-bold hover:bg-indigo-900 shadow-sm">Añadir al carrito</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    )}
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
```eof
