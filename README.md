# diagrama-abc
import plotly.graph_objects as go
import plotly.io as pio
import numpy as np

# ─── Datos actualizados (64 ítems) en orden EXACTAMENTE decreciente de DAV ──
codigos = [
    'CC-006', 'CC-007', 'CC-005', 'MP-003', 'CC-008', 'CC-002', 'MP-001',
    'MP-008', 'MP-002', 'MP-006', 'MP-004', 'CC-010', 'CC-001', 'CC-009',
    'CC-004', 'CC-011', 'CON-001', 'CC-003', 'EMB-001', 'MP-007',
    'EMB-002', 'EMB-005', 'CON-002', 'EMB-003', 'EMB-004',
    'REP-004', 'REP-005', 'REP-001', 'EPP-008', 'REP-008',
    'EPP-011', 'REP-007', 'REP-009', 'REP-019', 'EPP-006',
    'EPP-010', 'EPP-007', 'EPP-012', 'REP-010', 'REP-017',
    'REP-020', 'EPP-009', 'EPP-014', 'REP-016', 'REP-003',
    'REP-006', 'EPP-003', 'REP-014', 'REP-018', 'REP-002',
    'EPP-004', 'EPP-005', 'EPP-013', 'EPP-015', 'REP-015',
    'EPP-001', 'REP-022', 'EPP-016', 'REP-021', 'REP-013',
    'EPP-017', 'REP-012', 'REP-011', 'EPP-002'
]

dav = [
    510000, 225000, 165000, 141000, 135000, 120000, 114000,
    108000,  75000,  42000,  36000,  36000,  27000,  27000,
     24000,  24000,  18000,  15000,  15000,   6300,
      4500,   3600,   3000,   2400,   2250,
      1860,    492,    432,    377,    328.8,
      296.4,  267.3,  226.2,    220,     192,
      178.2,  171.5,  164.4,    164,     140,
       140,   123.4,  116.4,  115.2,   92.55,
      73.8,   68.4,     66,     66,    57.6,
      51.5,   49.4,   49.2,     48,      48,
        44,     42,   38.4,     36,   30.85,
      30.2,   30.2,   24.7,    9.6
]

# ─── Cálculos ABC ───────────────────────────────────────────────────────────
total = sum(dav)
porcentajes = [d / total * 100 for d in dav]
acumulado = np.cumsum(porcentajes)

n_A, n_B = 7, 8
colores_barras = ['#d9534f'] * n_A + ['#f0ad4e'] * n_B + ['#5cb85c'] * (len(codigos) - n_A - n_B)

# ─── Colores para el hover label (fondo de la tarjeta) ──────────────────────
hover_bgcolors = ['#d9534f'] * n_A + ['#f0ad4e'] * n_B + ['#5cb85c'] * (len(codigos) - n_A - n_B)
hover_fontcolors = ['#ffffff'] * n_A + ['#000000'] * (n_B + (len(codigos) - n_A - n_B))

# ─── Texto mejorado para el hover ───────────────────────────────────────────
hover_text = [
    f"<b>Código:</b> {cod}<br>"
    f"<b>DAV:</b> US$ {d:,.2f}<br>"
    f"<b>% Individual:</b> {p:.2f}%<br>"
    f"<b>% Acumulado:</b> {a:.1f}%"
    for cod, d, p, a in zip(codigos, dav, porcentajes, acumulado)
]

# ─── Crear figura ────────────────────────────────────────────────────────────
fig = go.Figure()

# ─── Agregar zonas sombreadas de fondo (A, B, C) ────────────────────────────
fig.add_hrect(y0=0, y1=80, fillcolor='red', opacity=0.06, line_width=0, yref='y2')
fig.add_hrect(y0=80, y1=95, fillcolor='yellow', opacity=0.10, line_width=0, yref='y2')
fig.add_hrect(y0=95, y1=100, fillcolor='green', opacity=0.06, line_width=0, yref='y2')

# ─── Barras con colores de clase ────────────────────────────────────────────
fig.add_trace(go.Bar(
    x=np.arange(len(codigos)),
    y=dav,
    marker=dict(
        color=colores_barras,
        line=dict(color='white', width=0.3)
    ),
    hovertemplate='%{customdata}<extra></extra>',
    customdata=hover_text,
    name='DAV (USD)',
    showlegend=False,
    hoverlabel=dict(
        bgcolor=hover_bgcolors,
        bordercolor='#333333',
        font=dict(
            size=12,
            family='Arial',
            color=hover_fontcolors
        )
    ),
    hoverinfo='text'
))

# ─── Capa invisible de puntos para facilitar la selección ───────────────────
fig.add_trace(go.Scatter(
    x=np.arange(len(codigos)),
    y=dav,
    mode='markers',
    marker=dict(
        size=25,
        color='rgba(0,0,0,0)',
        line=dict(width=0)
    ),
    hovertemplate='%{customdata}<extra></extra>',
    customdata=hover_text,
    showlegend=False,
    hoverlabel=dict(
        bgcolor=hover_bgcolors,
        bordercolor='#333333',
        font=dict(
            size=12,
            family='Arial',
            color=hover_fontcolors
        )
    ),
    hoverinfo='text'
))

# ─── Línea de % acumulado ───────────────────────────────────────────────────
fig.add_trace(go.Scatter(
    x=np.arange(len(codigos)),
    y=acumulado,
    mode='lines+markers',
    marker=dict(color='black', size=4, symbol='circle'),
    line=dict(color='black', width=2.5),
    name='% Acumulado',
    yaxis='y2',
    hovertemplate='<b>% Acumulado:</b> %{y:.1f}%<extra></extra>',
    hoverlabel=dict(
        bgcolor='white',
        bordercolor='#333333',
        font=dict(size=12, family='Arial', color='black')
    )
))

# ─── Líneas horizontales (80% y 95%) ────────────────────────────────────────
for y_val, color, label in [
    (80, '#CC0000', '  Clase A (80%)  '),
    (95, '#CC6600', '  Clase B (95%)  ')
]:
    fig.add_hline(
        y=y_val,
        line_dash='dash',
        line_color=color,
        line_width=2,
        opacity=0.7,
        yref='y2',
        annotation_text=label,
        annotation_position='right',
        annotation_font=dict(size=11, color=color, family='Arial'),
        annotation_bgcolor='rgba(255,255,255,0.8)'
    )

# ─── Anotación Clase C (fuera del gráfico) ──────────────────────────────────
fig.add_annotation(
    text='  Clase C (100%)  ',
    x=1.06,
    y=97.5,
    xref='paper',
    yref='y2',
    showarrow=False,
    font=dict(size=11, color='#006600', family='Arial'),
    bgcolor='rgba(255,255,255,0.8)'
)

# ─── Líneas verticales divisorias entre clases ──────────────────────────────
limite_AB = n_A - 0.5
limite_BC = n_A + n_B - 0.5
for x_pos in [limite_AB, limite_BC]:
    fig.add_vline(
        x=x_pos,
        line_dash='dash',
        line_color='gray',
        line_width=1.2,
        opacity=0.6,
        layer='below'
    )

# ─── Layout profesional ─────────────────────────────────────────────────────
fig.update_layout(
    title=dict(
        text='<b>Diagrama ABC de Insumos</b><br>'
             '<span style="font-size:14px;font-weight:normal">'
             'Bomba Centrífuga Monoblock 5 HP — 64 ítems</span>',
        x=0.5,
        xanchor='center'
    ),
    xaxis=dict(
        title='<b>Código de Ítem</b>',
        tickmode='array',
        tickvals=np.arange(len(codigos)),
        ticktext=codigos,
        tickangle=90,
        tickfont=dict(size=9, family='Arial'),
        fixedrange=True,
        showgrid=False
    ),
    yaxis=dict(
        title='<b>Demanda Anual Valorizada (USD)</b>',
        tickformat=',d',
        tickfont=dict(size=10, family='Arial'),
        fixedrange=True,
        showgrid=True,
        gridcolor='#EEEEEE'
    ),
    yaxis2=dict(
        title='<b>% Acumulado</b>',
        overlaying='y',
        side='right',
        range=[0, 105],
        ticksuffix='%',
        tickfont=dict(size=10, family='Arial'),
        fixedrange=True,
        showgrid=False
    ),
    hovermode='closest',
    template='plotly_white',
    width=1800,
    height=750,
    margin=dict(t=80, b=160, l=100, r=150),
    dragmode=False
)

# ─── Configuración interactiva ───────────────────────────────────────────────
config = {
    'displayModeBar': True,
    'modeBarButtonsToRemove': ['zoom2d', 'pan2d', 'select2d', 'lasso2d', 'zoomIn2d', 'zoomOut2d', 'autoScale2d', 'resetScale2d'],
    'displaylogo': False,
    'toImageButtonOptions': {
        'format': 'png',
        'filename': 'diagrama_abc',
        'height': 750,
        'width': 1800,
        'scale': 2
    }
}

# ─── Guardar ─────────────────────────────────────────────────────────────────
pio.write_html(
    fig,
    file='diagrama_abc_interactivo.html',
    auto_open=True,
    config=config
)
print("✅ Archivo 'diagrama_abc_interactivo.html' generado con datos ordenados correctamente.")
