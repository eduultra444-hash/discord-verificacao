require("dotenv").config();

const fs = require("fs");
const path = require("path");
const express = require("express");
const axios = require("axios");

const {
  Client,
  GatewayIntentBits,
  REST,
  Routes,
  SlashCommandBuilder,
  PermissionFlagsBits,
  EmbedBuilder,
  ButtonBuilder,
  ButtonStyle,
  ActionRowBuilder,
  AttachmentBuilder
} = require("discord.js");

const app = express();

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMembers
  ]
});

const PORT = Number(process.env.PORT || 3000);
const BASE_URL = (process.env.BASE_URL || `http://localhost:${PORT}`).replace(/\/+$/, "");
const REDIRECT_URI = `${BASE_URL}/callback`;

const commands = [
  new SlashCommandBuilder()
    .setName("painel")
    .setDescription("Envia o painel de verificação")
    .setDefaultMemberPermissions(PermissionFlagsBits.Administrator)
    .toJSON()
];

client.once("ready", async () => {
  console.log(`Bot conectado como ${client.user.tag}`);

  const rest = new REST({ version: "10" }).setToken(process.env.BOT_TOKEN);

  await rest.put(
    Routes.applicationGuildCommands(
      process.env.CLIENT_ID,
      process.env.GUILD_ID
    ),
    { body: commands }
  );

  console.log("Comando /painel registrado.");
  console.log(`Site: ${BASE_URL}`);
});

client.on("interactionCreate", async interaction => {
  if (!interaction.isChatInputCommand()) return;
  if (interaction.commandName !== "painel") return;

  const embed = new EmbedBuilder()
    .setColor(0x5865F2)
    .setTitle(`✅ ${process.env.PANEL_TITLE || "Verificação"}`)
    .setDescription(
      `**${process.env.SERVER_NAME || "Servidor"}**\n\n` +
      `${process.env.PANEL_MESSAGE || "Clique no botão abaixo para verificar sua conta."}`
    )
    .setFooter({
      text: process.env.PANEL_FOOTER || "Sistema de verificação"
    });

  const files = [];
  const serverImage = path.join(__dirname, "public", "server.png");

  if (fs.existsSync(serverImage)) {
    files.push(new AttachmentBuilder(serverImage, { name: "server.png" }));
    embed.setThumbnail("attachment://server.png");
  }

  const row = new ActionRowBuilder().addComponents(
    new ButtonBuilder()
      .setLabel("Verificar")
      .setEmoji("✅")
      .setStyle(ButtonStyle.Link)
      .setURL(BASE_URL)
  );

  await interaction.reply({
    embeds: [embed],
    components: [row],
    files
  });
});

app.use(express.static("public"));

function escapeHtml(text) {
  return String(text ?? "")
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#039;");
}

function avatarUrl(user) {
  if (!user.avatar) {
    const index = Number((BigInt(user.id) >> 22n) % 6n);
    return `https://cdn.discordapp.com/embed/avatars/${index}.png`;
  }
  const ext = user.avatar.startsWith("a_") ? "gif" : "png";
  return `https://cdn.discordapp.com/avatars/${user.id}/${user.avatar}.${ext}?size=256`;
}

app.get("/", (req, res) => {
  const params = new URLSearchParams({
    client_id: process.env.CLIENT_ID,
    response_type: "code",
    redirect_uri: REDIRECT_URI,
    scope: "identify"
  });

  const oauthUrl = `https://discord.com/oauth2/authorize?${params.toString()}`;

  res.send(`<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Verificação</title>
<link rel="stylesheet" href="/style.css">
</head>
<body>
<main class="card">
  <div style="font-size:62px">🛡️</div>
  <h1>${escapeHtml(process.env.SERVER_NAME || "Servidor")}</h1>
  <p>Verifique sua conta Discord para liberar seu acesso ao servidor.</p>
  <a class="button" href="${oauthUrl}">Entrar com Discord e verificar</a>
  <div class="small">🔒 Nunca pediremos sua senha ou token.</div>
</main>
</body>
</html>`);
});

app.get("/callback", async (req, res) => {
  if (!req.query.code) {
    return res.status(400).send("Código de autorização não recebido.");
  }

  try {
    const tokenResponse = await axios.post(
      "https://discord.com/api/oauth2/token",
      new URLSearchParams({
        client_id: process.env.CLIENT_ID,
        client_secret: process.env.CLIENT_SECRET,
        grant_type: "authorization_code",
        code: req.query.code,
        redirect_uri: REDIRECT_URI
      }),
      { headers: { "Content-Type": "application/x-www-form-urlencoded" } }
    );

    const userResponse = await axios.get(
      "https://discord.com/api/users/@me",
      {
        headers: {
          Authorization: `Bearer ${tokenResponse.data.access_token}`
        }
      }
    );

    const user = userResponse.data;
    const guild = await client.guilds.fetch(process.env.GUILD_ID);
    const member = await guild.members.fetch(user.id).catch(() => null);

    if (!member) {
      return res.status(403).send("Essa conta não está no servidor.");
    }

    const role = await guild.roles.fetch(process.env.VERIFIED_ROLE_ID);

    if (!role) {
      return res.status(500).send("Cargo Verificado não encontrado.");
    }

    if (!member.roles.cache.has(role.id)) {
      await member.roles.add(role, "Verificação concluída pelo site");
    }

    const name = escapeHtml(user.global_name || user.username);
    const username = escapeHtml(user.username);
    const avatar = avatarUrl(user);

    res.send(`<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Verificado</title>
<link rel="stylesheet" href="/style.css">
</head>
<body>
<main class="card">
  <img class="avatar" src="${avatar}">
  <div class="check">✓</div>
  <h1>Verificado!</h1>
  <div style="font-size:21px;font-weight:bold">${name}</div>
  <p>@${username}</p>
  <div class="success">✅ Seu acesso ao servidor foi liberado.</div>
  <div class="small">Você pode fechar esta página e voltar ao Discord.</div>
</main>
</body>
</html>`);
  } catch (err) {
    console.error(err.response?.data || err);
    res.status(500).send("Erro ao concluir a verificação.");
  }
});

app.listen(PORT, () => {
  console.log(`Site iniciado na porta ${PORT}`);
});

client.login(process.env.BOT_TOKEN);
