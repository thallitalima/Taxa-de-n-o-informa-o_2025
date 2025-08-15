# -*- coding: utf-8 -*-
# Requisitos: pandas, matplotlib
import pandas as pd
import matplotlib.pyplot as plt

# ------------------------------
# 1) Dados (tabela enviada)
# ------------------------------
data = {
    "Mês": ["Jun 2025","Mai 2025","Abr 2025","Mar 2025","Fev 2025","Jan 2025",
            "Dez 2024","Nov 2024","Out 2024","Set 2024","Ago 2024"],
    "Sem local": [0.48,0.49,0.51,0.53,0.55,0.57,0.59,0.61,0.64,0.66,0.71],
    "Sem status": [2.38,2.45,2.56,2.66,2.75,2.85,2.97,3.37,3.51,3.32,2.84],
    "Sem data": [4.99,5.15,4.60,4.52,4.40,5.70,6.53,7.06,6.39,5.98,6.74],
    "Sem operador": [26.84,26.72,25.83,23.94,24.18,23.36,22.26,21.47,20.13,18.27,15.96],
    "Sem fornecedor": [78.38,78.19,77.75,76.86,76.92,76.35,77.15,77.61,77.32,77.08,None],
    "Sem software": [95.72,95.83,95.65,95.48,95.33,95.16,94.96,95.09,94.89,95.02,None],
    "Sem custo": [67.93,67.89,68.29,67.55,66.48,66.10,65.88,65.34,65.81,65.12,64.89],
    "Nº de projetos ativos": [421,408,391,376,364,351,337,326,313,301,282]
}
df = pd.DataFrame(data)

# Observação: as “taxas ponderadas” são, por definição, a razão
# (casos sem info / projetos ativos no mês). Como os valores fornecidos
# já estão em %, não há “acúmulo”. Ainda assim, manteremos a
# transformação explícita para deixar claro o método.

categorias = ["Sem local", "Sem status", "Sem data", "Sem operador",
              "Sem fornecedor", "Sem software", "Sem custo"]

df_ponderado = df.copy()
for cat in categorias:
    # Converte % -> contagem e volta para % sobre o mesmo denominador do mês
    # (serve para documentar o cálculo e evitar dúvidas sobre acúmulo)
    contagem = (df[cat] / 100.0) * df["Nº de projetos ativos"]
    df_ponderado[cat] = (contagem / df["Nº de projetos ativos"]) * 100.0

# ------------------------------
# 2) Funções de plot
# ------------------------------
def grafico_combinado_correlacionada(df_plot, categorias_plot, titulo, cor_proj="#9370DB"):
    """
    Linhas = taxas (eixo esquerdo)
    Barras = nº de projetos ativos (eixo direito, mesma cor do eixo)
    """
    fig, ax1 = plt.subplots(figsize=(10, 6))

    # Linhas (taxas ponderadas)
    for cat in categorias_plot:
        ax1.plot(df_plot["Mês"], df_plot[cat], marker="o", label=cat)
    ax1.set_xlabel("Mês")
    ax1.set_ylabel("Taxa de não informação ponderada (%)")
    ax1.invert_xaxis()                # cronologia: esquerda = mais antigo
    ax1.grid(True, linestyle="--", alpha=0.5)

    # Barras + eixo direito (mesma cor)
    ax2 = ax1.twinx()
    ax2.bar(df_plot["Mês"], df_plot["Nº de projetos ativos"], alpha=0.30,
            color=cor_proj, label="Projetos ativos")
    ax2.set_ylabel("Nº de projetos ativos", color=cor_proj)
    ax2.tick_params(axis="y", colors=cor_proj)

    # Título e legenda
    fig.suptitle(titulo, fontsize=14)
    # legenda combinada (linhas + barras)
    linhas, labels_linhas = ax1.get_legend_handles_labels()
    barras, labels_barras = ax2.get_legend_handles_labels()
    fig.legend(linhas + barras, labels_linhas + labels_barras,
               loc="upper center", bbox_to_anchor=(0.5, -0.05), ncol=4)

    plt.xticks(rotation=45)
    plt.tight_layout()
    return fig

# ------------------------------
# 3) Gráficos
# ------------------------------
# (A) Melhoras
fig1 = grafico_combinado_correlacionada(
    df_ponderado,
    ["Sem local", "Sem status", "Sem data"],
    "Melhora nas Taxas de Não Informação vs Projetos Ativos (eixo lilás)",
    cor_proj="#9370DB"  # lilás
)

# (B) Piora / manutenção
fig2 = grafico_combinado_correlacionada(
    df_ponderado,
    ["Sem operador", "Sem fornecedor", "Sem software", "Sem custo"],
    "Aumento ou Manutenção das Taxas de Não Informação vs Projetos Ativos (eixo lilás)",
    cor_proj="#9370DB"  # lilás
)

plt.show()

# ------------------------------
# 4) (Opcional) salvar as figuras
# ------------------------------
# fig1.savefig("melhoras_vs_projetos_ativos.png", dpi=300, bbox_inches="tight")
# fig2.savefig("piora_vs_projetos_ativos.png", dpi=300, bbox_inches="tight")
# Taxa-de-n-o-informa-o_2025
