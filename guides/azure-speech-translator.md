# Azure Speech + Translator setup

MiraSpeak's **Azure Speech** engine and **Azure Translator** provider each need an Azure resource. This guide creates one resource group with both, then puts the keys into MiraSpeak.

- Speech and Translator are **separate resources** with separate keys. A key from one won't work for the other.
- Both have a free tier (F0). One free resource per service per subscription.
- MiraSpeak keeps keys in your OS credential store (keyring), never in `settings.toml`.

> **Security note:** a key lets anyone bill your subscription. Never commit it, paste it in chat, or show it on stream. If it leaks, regenerate it (see [Key rotation](#key-rotation)).

## 1. Azure account and subscription

Sign up at [azure.microsoft.com/free](https://azure.microsoft.com/free/). New accounts get trial credit, and the F0 tiers below are free on any subscription. A card is required for identity checks. When the trial ends you choose pay-as-you-go to keep using S0/S1.

## 2. Create a resource group

In the [Azure portal](https://portal.azure.com/): search **Resource groups → Create**.

| Field | Value |
| --- | --- |
| Subscription | your subscription |
| Resource group | for example, `<your-resource-group>` (`miraspeak`) |
| Region | `<region>` close to you, for example, `eastus`, `westeurope`, `japaneast` |

Tip: create Speech in this same region. Not every voice/model is in every region; check [Speech region support](https://learn.microsoft.com/azure/ai-services/speech-service/regions) first if you want HD voices. Deleting the group later removes everything in it.

## 3. Create the Azure AI Speech resource

Search **Speech** in the portal and create a Speech service resource (the portal may list it under Foundry Tools / Azure AI services).

| Field | Value |
| --- | --- |
| Resource group | `<your-resource-group>` |
| Region | `<region>` (same as above) |
| Name | unique, for example, `<your-speech-name>` |
| Pricing tier | **Free F0** to try, **Standard S0** for regular use |

F0 has a small monthly text-to-speech character allowance and lower rate limits; S0 is billed per character (HD/neural voices cost more). See [Speech pricing](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

After deploy: **Go to resource → Resource Management → Keys and Endpoint.** Copy **Key 1** (Key 2 is a spare) and the **Location/Region** (for example, `eastus`). MiraSpeak needs the key and region, not the endpoint URL.

## 4. Create the Azure AI Translator resource

Search **Translator** and create a Translator resource.

| Field | Value |
| --- | --- |
| Resource group | `<your-resource-group>` |
| Region | **Global** (simplest) or a specific `<region>` |
| Name | unique, for example, `<your-translator-name>` |
| Pricing tier | **Free F0** (a character allowance that resets each month) or **Standard S1** (pay per character) |

Then **Keys and Endpoint** again: copy **Key 1** and note the **Location/Region**.

- For a Global resource, leave Region blank. MiraSpeak calls the global endpoint `api.cognitive.microsofttranslator.com`.
- For a regional or multi-service resource, enter the region in MiraSpeak. The same global endpoint works, but the region must be sent with the key.

See [Create a Translator resource](https://learn.microsoft.com/azure/ai-services/translator/how-to/create-translator-resource) and [Translator pricing](https://azure.microsoft.com/pricing/details/cognitive-services/translator/).

## 5. Add the keys to MiraSpeak

Open **Voice engines → Cloud** and set **Engine** to **Azure Speech**.

1. Paste the key into **Speech key**, select **Save key**.
2. Enter **Region** (for example, `eastus`).
3. **Test connection**, then **Refresh voices**.

Open **Translation → Cloud** and set **Engine** to **Azure Translator**.

1. Paste the key into **Key**, select **Save key**.
2. **Region**: leave blank for a Global resource; enter `<region>` for regional or multi-service resources.
3. Select **Test**.
4. On **Translation → General**, choose Azure Translator as the provider.

Saved keys aren't shown again. Use **Clear key** to remove one.

## Cost and quota

- The **Usage** meter on the Azure Speech page is a local, approximate character count, not portal billing. Set **Limit** and **At limit** there. At limit can be **Continue**, **Hard stop**, or **Play from cache if available**.
- Translation cache hits are counted on the Azure Translator page and don't hit Azure.
- For real numbers use the portal: **Cost Management → Budgets** (set an alert) and the resource's **Metrics**.
- F0 quotas reset monthly. When a quota runs out you get `429` or `403` until the reset, or until you move to S0 (Speech) or S1 (Translator).

## Key rotation

Each resource has two keys so you can rotate without downtime:

1. Switch MiraSpeak to **Key 2** and save.
2. In **Keys and Endpoint**, **Regenerate Key 1**.
3. Next time, swap back the other way.

If a key leaks, regenerate it immediately; the old one stops working. Keep keys out of git, screenshots, logs, and stream overlays.

## Troubleshooting

| Symptom | What to do |
| --- | --- |
| `401` on Speech | Check **Speech key** and **Region** on Voice engines → Cloud. The region must match the Speech resource. |
| `401` on Translator | Check **Key** and **Region** on Translation → Cloud. A regional resource needs its region. |
| `429` | Wait for the rate limit or the F0 monthly reset, or move to S0 (Speech) or S1 (Translator). |
| `403` quota exceeded | Wait for the monthly reset, or check that the subscription is enabled and has credit. |
| Voice missing, or HD voices fail to synthesize | Use a Speech region that offers that model, or pick another model on the **Engine** chip. |
| Translator key works in one app, not MiraSpeak | Paste the Translator key on Translation → Cloud. The Speech key doesn't work there, and the Translator key doesn't work for Speech. |
