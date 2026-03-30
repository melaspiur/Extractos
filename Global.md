import re, tkinter as tk
from tkinter import filedialog, ttk, messagebox
import pdfplumber, pandas as pd

def parse_generico(t):
    pat=r"^(\d{2}/\d{2}/\d{2})\s+(.+?)\s+(\S+)\s+([\d\.,]+)\s+([\d\.,]+)$"
    return [re.match(pat," ".join(l.split())).groups() for l in t.splitlines() if re.match(pat," ".join(l.split()))]

def parse_bna(t):
    pat=r"^(\d{2}/\d{2}/\d{2})\s+(.+?)\s+(\d+)\s+([\d\.,]+)\s+([\d\.,]+)$"
    return [re.match(pat," ".join(l.split())).groups() for l in t.splitlines() if re.match(pat," ".join(l.split()))]

def procesar():
    pdf=filedialog.askopenfilename(filetypes=[("PDF","*.pdf")])
    if not pdf: return
    motor=combo.get()
    txt=""
    with pdfplumber.open(pdf) as p:
        for pg in p.pages: txt += (pg.extract_text() or "") + "\n"
    rows = parse_bna(txt) if motor=="Banco Nación" else parse_generico(txt)
    df=pd.DataFrame(rows,columns=["fecha","movimiento","comprobante","importe","saldo"])
    xlsx=filedialog.asksaveasfilename(defaultextension=".xlsx")
    if xlsx: df.to_excel(xlsx,index=False); messagebox.showinfo("OK","Archivo generado")

root=tk.Tk(); root.title("PDF a Excel")
combo=ttk.Combobox(root,values=["Genérico","Banco Nación"]); combo.set("Banco Nación"); combo.pack(padx=20,pady=10)
ttk.Button(root,text="Seleccionar PDF / Procesar / Exportar",command=procesar).pack(padx=20,pady=20)
root.mainloop()
