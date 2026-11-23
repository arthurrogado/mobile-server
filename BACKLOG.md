# Backlog — mobile-server fork (arthurrogado/mobile-server)

> Rodapé: itens levantados na sessão 2026-11-23 (S167, wiki `tecnico_ti`). Ordem de prioridade aproximada.

## Corrigido
- [x] Context window travado em 4096 → bump para 8192 (`LlmChatModelHelper.kt`, `Consts.kt`, `model_allowlist.json`) — validado com prompt de ~7000 tokens.
- [x] Stream OpenAI sem `finish_reason` → OpenCode rejeitava com `Invalid ... stream event`; adicionado chunk terminal `finish_reason: "stop"` em chat e completions.

## Backlog (RFs do spec `tecnico_ti/specs/021-mobile-server-evolucao/`)
- [ ] RF-01 — Preservar modelos baixados entre versões do app. **Lição operacional**: entre builds do MESMO fork (mesma chave de debug do GitHub Actions), usar `adb install -r` (mantém dados). `adb uninstall` só quando a assinatura muda de verdade (fork ↔ app original `techjarves`), porque apaga todos os modelos. Na sessão atual isso aconteceu por descuido e o Gemma teve de ser rebaixado.
- [ ] RF-01b — Detectar/importar modelos `.task` de `com.google.ai.edge.gallery` e do fork anterior no primeiro boot.
- [ ] RF-02 — Catálogo de modelos oficiais do Google Edge Gallery no allowlist.
- [ ] RF-03 — Editor de parâmetros na tela de carregamento: context window (slider + input), topK, topP, temperatura; reinicializar `EngineConfig` ao alterar.
- [ ] RF-04 — Estimativa de memória (modelo + KV cache) x RAM disponível, com indicador verde→amarelo→vermelho de "tensão" do dispositivo.
- [ ] RF-05 — Offload de KV cache para disco (se o LiteRT-LM suportar) + indicador visual; se não houver API, registrar limitação.

## Pendente de validação (sessão atual)
- [ ] Confirmar que o stream com `finish_reason` corrigido aceita o "oi" na TUI do OpenCode sem erro de stream event.

## Notas
- Não root no celular; GPU via NNAPI/GPU delegate nativo do Google (llama.cpp/OpenCL no Termux é bloqueado por SELinux no Android 16).
- Build: GitHub Actions (`.github/workflows/build_android.yaml`) + `gh` com escopo `workflow` + `actions/upload-artifact@v4`.
- Instalação: ADB wireless (`adb pair` + `adb connect`).

## Descobertas da sessão (continuação)
- [ ] **Estabilidade do servidor em background**: o app descarrega o modelo e para o servidor quando sai de foreground ou quando a conexão ADB/Wi-Fi oscila. `ModelKeepAliveService` e `OpenAiServerService` aparecem como `isForeground=true` no dumpsys, mas a porta 8080 deixa de escutar. Investigar: (a) o servidor só sobe se o modelo estiver carregado? (b) o `model.instância` é perdido ao trocar de aba? (c) adicionar reconexão automática do servidor quando o modelo for recarregado.
- [ ] **Wake lock**:desabilitei o screen-off timeout via ADB (`settings put system screen_off_timeout 2147483647`), mas isso não impediu a queda. O Termux SSH também caiu (`NoValidConnectionsError`). Considerar `termux-wake-lock` via SSH estável ou Pedra/party.
- [ ] **ADB wireless instability**: o IP de pareamento `100.95.134.90:37017` caiu (`connection refused`) após ~30 min. O IP da Wi-Fi é `192.168.15.7`. Reconectar via `adb pair` se necessário.
- [x] Context 16384 no allowlist + slider (2000–32000) com input numérico + reinicialização automática do engine ao mudar (já implementado no fork).
