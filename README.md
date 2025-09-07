import React, { useEffect, useMemo, useState } from "react";
import { motion } from "framer-motion";
import { Search, Plus, Save, Upload, Download, ExternalLink, Info, ShoppingCart, Trash2, Link as LinkIcon, Image as ImageIcon, RefreshCw } from "lucide-react";

// --- Utilidades ---
const lsKey = "hierbas_store_v1";
const sampleHerbs = [
  {
    id: crypto.randomUUID(),
    nombre: "Canela (Cinnamomum verum)",
    precio: 2500,
    unidad: "100 g",
    stock: 15,
    imagen: "https://upload.wikimedia.org/wikipedia/commons/thumb/6/6f/Cinnamomum_verum_spices.jpg/640px-Cinnamomum_verum_spices.jpg",
    etiquetas: ["digestiva", "aromática"],
    descripcion: "Corteza aromática tradicionalmente usada en infusiones y repostería.",
    propiedades: [
      "Tradicionalmente usada para apoyar la digestión",
      "Saborizante natural en bebidas y postres",
    ],
    preparacion: "Infusión: 1/2 cucharadita por taza, agua caliente 90ºC, 8–10 min.",
    usoCulinario: "Infusiones, compotas, arroz con leche, curry, pastelería.",
    contra: "No usar en exceso durante el embarazo. Personas con alergia a la canela deben evitar su consumo.",
    links: [
      { titulo: "Wikipedia: Canela", url: "https://es.wikipedia.org/wiki/Canela" },
    ],
  },
  {
    id: crypto.randomUUID(),
    nombre: "Flor de Jamaica (Hibiscus sabdariffa)",
    precio: 2200,
    unidad: "100 g",
    stock: 20,
    imagen: "https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Hibiscus_sabdariffa_dried_fruit.jpg/640px-Hibiscus_sabdariffa_dried_fruit.jpg",
    etiquetas: ["infusión", "refrescante"],
    descripcion: "Cáliz seco de hibisco, ideal para agua de Jamaica y tés frutales.",
    propiedades: [
      "Tradicionalmente usada como refrescante e hidratante",
      "Aporta color y sabor ácido natural",
    ],
    preparacion: "Infusión: 1 cda por taza, 95ºC, 10 minutos. Endulzar a gusto.",
    usoCulinario: "Aguas saborizadas, mermeladas, salsas para carnes, repostería.",
    contra: "No exceder el consumo si hay hipotensión. Consulte si toma fármacos antihipertensivos.",
    links: [
      { titulo: "Wikipedia: Hibiscus sabdariffa", url: "https://es.wikipedia.org/wiki/Hibiscus_sabdariffa" },
    ],
  },
  {
    id: crypto.randomUUID(),
    nombre: "Laurel (Laurus nobilis)",
    precio: 1500,
    unidad: "25 g",
    stock: 30,
    imagen: "https://upload.wikimedia.org/wikipedia/commons/thumb/8/86/Laurus_nobilis_BotGardBln0906b.jpg/640px-Laurus_nobilis_BotGardBln0906b.jpg",
    etiquetas: ["culinaria", "aromática"],
    descripcion: "Hojas aromáticas para guisos, sopas y adobos.",
    propiedades: [
      "Tradicionalmente usada para saborizar platos",
      "Aporta aroma balsámico",
    ],
    preparacion: "Usar 1–2 hojas por preparación; retirar antes de servir.",
    usoCulinario: "Caldos, legumbres, estofados, carnes, encurtidos.",
    contra: "No consumir hojas enteras. Posibles alergias en personas sensibles.",
    links: [
      { titulo: "Wikipedia: Laurus nobilis", url: "https://es.wikipedia.org/wiki/Laurus_nobilis" },
    ],
  },
];

function useLocalState(key, initial) {
  const [state, setState] = useState(() => {
    try {
      const s = localStorage.getItem(key);
      return s ? JSON.parse(s) : initial;
    } catch {
      return initial;
    }
  });
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(state));
  }, [key, state]);
  return [state, setState];
}

export default function App() {
  const [items, setItems] = useLocalState(lsKey, sampleHerbs);
  const [q, setQ] = useState("");
  const [editing, setEditing] = useState(null);
  const [showForm, setShowForm] = useState(false);

  const filtered = useMemo(() => {
    const t = q.trim().toLowerCase();
    if (!t) return items;
    return items.filter((x) =>
      [
        x.nombre,
        x.descripcion,
        x.etiquetas?.join(" "),
        x.propiedades?.join(" "),
      ]
        .join(" ")
        .toLowerCase()
        .includes(t)
    );
  }, [items, q]);

  function resetForm() {
    setEditing({
      id: crypto.randomUUID(),
      nombre: "",
      precio: 0,
      unidad: "100 g",
      stock: 0,
      imagen: "",
      etiquetas: [],
      descripcion: "",
      propiedades: [],
      preparacion: "",
      usoCulinario: "",
      contra: "",
      links: [],
    });
  }

  useEffect(() => {
    if (showForm && !editing) resetForm();
  }, [showForm]);

  function saveItem() {
    if (!editing?.nombre) return alert("Escribe un nombre");
    setItems((prev) => {
      const exists = prev.some((p) => p.id === editing.id);
      return exists ? prev.map((p) => (p.id === editing.id ? editing : p)) : [editing, ...prev];
    });
    setShowForm(false);
    setEditing(null);
  }

  function removeItem(id) {
    if (!confirm("¿Eliminar este producto?")) return;
    setItems((prev) => prev.filter((x) => x.id !== id));
  }

  function exportJSON() {
    const blob = new Blob([JSON.stringify(items, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "hierbas_data.json";
    a.click();
    URL.revokeObjectURL(url);
  }

  function importJSON(e) {
    const file = e.target.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try {
        const data = JSON.parse(reader.result);
        if (!Array.isArray(data)) throw new Error("Formato inválido");
        setItems(data);
      } catch (err) {
        alert("Archivo inválido: " + err.message);
      }
    };
    reader.readAsText(file);
  }

  async function buscarInfo(nombre) {
    if (!nombre) return alert("Escribe un nombre");
    const t = nombre
      .replace(/\(.+?\)/g, "")
      .replace(/\s+/g, " ")
      .trim();

    const titulo = encodeURIComponent(t);
    try {
      // 1) Wikipedia resumen (ES)
      const sum = await fetch(
        `https://es.wikipedia.org/api/rest_v1/page/summary/${titulo}`
      ).then((r) => r.ok ? r.json() : null);

      // 2) Imagen en Wikimedia Commons (búsqueda simple)
      const commons = await fetch(
        `https://commons.wikimedia.org/w/api.php?action=query&generator=search&gsrsearch=${titulo}&gsrlimit=1&prop=imageinfo&iiprop=url&format=json&origin=*`
      ).then((r) => r.ok ? r.json() : null);

      const page = commons?.query?.pages
        ? Object.values(commons.query.pages)[0]
        : null;
      const imageUrl = page?.imageinfo?.[0]?.url || editing?.imagen || "";

      setEditing((prev) => ({
        ...prev,
        descripcion:
          prev.descripcion?.length > 10
            ? prev.descripcion
            : sum?.extract || prev.descripcion,
        imagen: imageUrl,
        links: [
          ...(prev.links || []),
          sum?.content_urls?.desktop?.page
            ? { titulo: `Wikipedia: ${t}`, url: sum.content_urls.desktop.page }
            : null,
        ].filter(Boolean),
      }));
    } catch (e) {
      alert("No se pudo buscar información automáticamente. Puedes completarla manualmente.");
      console.error(e);
    }
  }

  return (
    <div className="min-h-screen bg-neutral-50 text-neutral-800">
      {/* Encabezado */}
      <header className="sticky top-0 z-10 bg-white/80 backdrop-blur border-b border-neutral-200">
        <div className="max-w-6xl mx-auto px-4 py-4 flex items-center gap-3">
          <motion.div initial={{ opacity: 0, y: -8 }} animate={{ opacity: 1, y: 0 }} className="flex-1">
            <h1 className="text-2xl sm:text-3xl font-bold">Hierbas de Ángel</h1>
            <p className="text-sm text-neutral-500">Tienda editable para vender hierbas (Chile) – agrega productos, genera fichas, exporta datos.</p>
          </motion.div>
          <div className="hidden sm:flex items-center gap-2">
            <button onClick={() => { setShowForm(true); resetForm(); }} className="px-3 py-2 rounded-2xl shadow-sm bg-emerald-600 text-white flex items-center gap-2">
              <Plus size={18}/> Nuevo
            </button>
            <button onClick={exportJSON} className="px-3 py-2 rounded-2xl shadow-sm bg-neutral-800 text-white flex items-center gap-2"><Download size={18}/> Exportar</button>
            <label className="px-3 py-2 rounded-2xl shadow-sm bg-neutral-200 text-neutral-800 flex items-center gap-2 cursor-pointer">
              <Upload size={18}/> Importar JSON
              <input type="file" accept="application/json" onChange={importJSON} className="hidden"/>
            </label>
          </div>
        </div>
        <div className="max-w-6xl mx-auto px-4 pb-4">
          <div className="flex items-center gap-2 bg-white rounded-2xl border border-neutral-200 px-3 py-2 shadow-sm">
            <Search size={18} className="opacity-70"/>
            <input value={q} onChange={(e)=>setQ(e.target.value)} placeholder="Buscar por nombre, etiqueta o descripción…" className="w-full outline-none py-1"/>
          </div>
        </div>
      </header>

      {/* Aviso legal */}
      <div className="max-w-6xl mx-auto px-4 mt-4">
        <div className="rounded-2xl border border-amber-200 bg-amber-50 p-4 text-amber-900 flex gap-3">
          <Info className="shrink-0"/>
          <p className="text-sm leading-relaxed">
            <strong>Información general:</strong> Las propiedades descritas son <em>usos tradicionales</em> y no sustituyen el consejo médico. No se hacen afirmaciones de curación. Consulte a su médico si está embarazada, en lactancia, con tratamientos o condiciones específicas.
          </p>
        </div>
      </div>

      {/* Listado */}
      <main className="max-w-6xl mx-auto px-4 py-6 grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        {filtered.map((it) => (
          <motion.article key={it.id} initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }} className="bg-white rounded-2xl border border-neutral-200 shadow-sm overflow-hidden flex flex-col">
            <div className="aspect-[4/3] bg-neutral-100 overflow-hidden">
              {it.imagen ? (
                <img src={it.imagen} alt={it.nombre} className="w-full h-full object-cover"/>
              ) : (
                <div className="w-full h-full grid place-items-center text-neutral-400"><ImageIcon/></div>
              )}
            </div>
            <div className="p-4 flex flex-col gap-2">
              <h3 className="font-semibold text-lg">{it.nombre}</h3>
              <div className="text-sm text-neutral-600 line-clamp-3">{it.descripcion}</div>
              <div className="flex flex-wrap gap-2 py-1">
                {it.etiquetas?.map((t, i) => (
                  <span key={i} className="text-xs px-2 py-1 bg-emerald-50 text-emerald-700 rounded-full border border-emerald-200">{t}</span>
                ))}
              </div>
              <div className="flex items-center justify-between pt-1">
                <div className="font-semibold">${it.precio?.toLocaleString?.("es-CL") || it.precio} <span className="text-xs text-neutral-500">/ {it.unidad}</span></div>
                <div className="text-xs text-neutral-500">Stock: {it.stock}</div>
              </div>
              <div className="grid grid-cols-2 gap-2 text-sm mt-2">
                <button onClick={() => { setEditing(it); setShowForm(true); }} className="px-3 py-2 rounded-xl border border-neutral-200">Editar</button>
                <button onClick={() => removeItem(it.id)} className="px-3 py-2 rounded-xl border border-red-200 text-red-700 flex items-center gap-1 justify-center"><Trash2 size={16}/>Eliminar</button>
              </div>
            </div>
          </motion.article>
        ))}
      </main>

      {/* Pie */}
      <footer className="max-w-6xl mx-auto px-4 pb-20">
        <div className="bg-white border border-neutral-200 rounded-2xl p-4 text-sm text-neutral-600">
          <p className="mb-1">© {new Date().getFullYear()} Hierbas de Ángel. Venta de hierbas e infusiones artesanales en Chile.</p>
          <p>Incluye exportación/importación de catálogo (JSON) y búsqueda automática en Wikipedia/Wikimedia para agilizar tus fichas.</p>
        </div>
      </footer>

      {/* Botón flotante nuevo */}
      <button
        onClick={() => { setShowForm(true); resetForm(); }}
        className="fixed bottom-6 right-6 px-4 py-3 rounded-full shadow-lg bg-emerald-600 text-white flex items-center gap-2"
      >
        <Plus size={18}/> Agregar producto
      </button>

      {/* Modal formulario */}
      {showForm && (
        <div className="fixed inset-0 bg-black/30 backdrop-blur-sm grid place-items-center p-4 z-20">
          <div className="w-full max-w-2xl bg-white rounded-2xl shadow-xl border border-neutral-200 overflow-hidden">
            <div className="p-4 border-b flex items-center justify-between">
              <h2 className="font-semibold">{editing?.id && items.some(x=>x.id===editing.id) ? "Editar" : "Nuevo"} producto</h2>
              <button onClick={()=>{setShowForm(false); setEditing(null);}} className="text-neutral-500">Cerrar ✕</button>
            </div>
            <div className="p-4 grid gap-3">
              <div className="grid sm:grid-cols-2 gap-3">
                <label className="grid gap-1">
                  <span className="text-sm">Nombre común / científico</span>
                  <input className="px-3 py-2 rounded-xl border" value={editing?.nombre||""} onChange={(e)=>setEditing({...editing, nombre:e.target.value})} placeholder="Ej: Flor de Jamaica (Hibiscus sabdariffa)"/>
                </label>
                <label className="grid gap-1">
                  <span className="text-sm">Precio</span>
                  <input type="number" className="px-3 py-2 rounded-xl border" value={editing?.precio||0} onChange={(e)=>setEditing({...editing, precio:Number(e.target.value)})} />
                </label>
              </div>
              <div className="grid sm:grid-cols-3 gap-3">
                <label className="grid gap-1">
                  <span className="text-sm">Unidad</span>
                  <input className="px-3 py-2 rounded-xl border" value={editing?.unidad||""} onChange={(e)=>setEditing({...editing, unidad:e.target.value})} placeholder="Ej: 100 g, 50 g"/>
                </label>
                <label className="grid gap-1">
                  <span className="text-sm">Stock</span>
                  <input type="number" className="px-3 py-2 rounded-xl border" value={editing?.stock||0} onChange={(e)=>setEditing({...editing, stock:Number(e.target.value)})} />
                </label>
                <label className="grid gap-1">
                  <span className="text-sm">Imagen (URL)</span>
                  <input className="px-3 py-2 rounded-xl border" value={editing?.imagen||""} onChange={(e)=>setEditing({...editing, imagen:e.target.value})} placeholder="Pega una URL pública (Wikimedia, tu hosting, etc.)"/>
                </label>
              </div>
              <label className="grid gap-1">
                <span className="text-sm">Descripción corta</span>
                <textarea className="px-3 py-2 rounded-xl border" rows={3} value={editing?.descripcion||""} onChange={(e)=>setEditing({...editing, descripcion:e.target.value})}/>
              </label>
              <div className="grid sm:grid-cols-2 gap-3">
                <label className="grid gap-1">
                  <span className="text-sm">Propiedades (una por línea, usa lenguaje de uso tradicional)</span>
                  <textarea className="px-3 py-2 rounded-xl border" rows={4} value={(editing?.propiedades||[]).join("\n")} onChange={(e)=>setEditing({...editing, propiedades:e.target.value.split(/\n+/).map(s=>s.trim()).filter(Boolean)})}/>
                </label>
                <div className="grid gap-3">
                  <label className="grid gap-1">
                    <span className="text-sm">Preparación de infusión</span>
                    <input className="px-3 py-2 rounded-xl border" value={editing?.preparacion||""} onChange={(e)=>setEditing({...editing, preparacion:e.target.value})} placeholder="Ej: 1 cda por taza, 95ºC, 10 min"/>
                  </label>
                  <label className="grid gap-1">
                    <span className="text-sm">Uso culinario</span>
                    <input className="px-3 py-2 rounded-xl border" value={editing?.usoCulinario||""} onChange={(e)=>setEditing({...editing, usoCulinario:e.target.value})} placeholder="Platos o bebidas donde usarla"/>
                  </label>
                  <label className="grid gap-1">
                    <span className="text-sm">Contraindicaciones (precauciones)</span>
                    <input className="px-3 py-2 rounded-xl border" value={editing?.contra||""} onChange={(e)=>setEditing({...editing, contra:e.target.value})} placeholder="Ej: Embarazo, interacciones, alergias"/>
                  </label>
                </div>
              </div>
              <label className="grid gap-1">
                <span className="text-sm">Etiquetas (separadas por coma)</span>
                <input className="px-3 py-2 rounded-xl border" value={(editing?.etiquetas||[]).join(", ")} onChange={(e)=>setEditing({...editing, etiquetas:e.target.value.split(",").map(s=>s.trim()).filter(Boolean)})} placeholder="digestiva, aromática, infusión"/>
              </label>

              <div className="grid gap-2">
                <span className="text-sm">Links externos</span>
                {(editing?.links||[]).map((l, i) => (
                  <div key={i} className="grid grid-cols-12 gap-2">
                    <input className="col-span-4 px-3 py-2 rounded-xl border" value={l.titulo} onChange={(e)=>{
                      const links = [...editing.links];
                      links[i] = { ...links[i], titulo: e.target.value };
                      setEditing({ ...editing, links });
                    }} placeholder="Título"/>
                    <input className="col-span-8 px-3 py-2 rounded-xl border" value={l.url} onChange={(e)=>{
                      const links = [...editing.links];
                      links[i] = { ...links[i], url: e.target.value };
                      setEditing({ ...editing, links });
                    }} placeholder="https://..."/>
                  </div>
                ))}
                <div className="flex gap-2">
                  <button onClick={()=>setEditing({...editing, links:[...(editing?.links||[]), {titulo:"", url:""}]})} className="px-3 py-2 rounded-xl border flex items-center gap-2"><LinkIcon size={16}/>Añadir link</button>
                  <button onClick={()=>buscarInfo(editing?.nombre)} className="px-3 py-2 rounded-xl border flex items-center gap-2"><RefreshCw size={16}/> Buscar info automática</button>
                </div>
              </div>

              {/* Botones */}
              <div className="flex flex-wrap gap-2 pt-2">
                <button onClick={saveItem} className="px-4 py-2 rounded-2xl bg-emerald-600 text-white flex items-center gap-2"><Save size={18}/> Guardar</button>
                <button onClick={()=>{
                  const ficha = `# ${editing?.nombre}\n\n**Precio:** $${editing?.precio?.toLocaleString?.("es-CL")}/${editing?.unidad}\n\n**Descripción:** ${editing?.descripcion}\n\n**Propiedades (uso tradicional):**\n- ${(editing?.propiedades||[]).join("\n- ")}\n\n**Infusión:** ${editing?.preparacion}\n\n**Uso culinario:** ${editing?.usoCulinario}\n\n**Precauciones:** ${editing?.contra}\n\n**Enlaces:**\n${(editing?.links||[]).map(l=>`- [${l.titulo}](${l.url})`).join("\n")}`;
                  navigator.clipboard.writeText(ficha);
                  alert("Ficha generada y copiada al portapapeles (Markdown)");
                }} className="px-4 py-2 rounded-2xl border flex items-center gap-2"><Info size={18}/> Generar ficha (MD)</button>
                <a href={`https://wa.me/?text=${encodeURIComponent(`Mira ${editing?.nombre} en mi tienda: ${location.href}`)}`} target="_blank" rel="noreferrer" className="px-4 py-2 rounded-2xl border flex items-center gap-2"><ExternalLink size={18}/> Compartir WhatsApp</a>
                <a href={`https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(location.href)}`} target="_blank" rel="noreferrer" className="px-4 py-2 rounded-2xl border flex items-center gap-2"><ExternalLink size={18}/> Compartir Facebook</a>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
