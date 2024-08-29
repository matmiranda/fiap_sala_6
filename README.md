using System.Text;
using System.Text.Json;

class Program
{
    static async Task Main(string[] args)
    {
        string grupo = "sala 6";
        string resposta = string.Empty;

        for (int i = 0; i < 100; i++) // Tenta até 100 vezes
        {
            string senha = GerarSenha();
            Console.WriteLine($"Tentativa {i + 1}: Senha gerada: {senha}");
            resposta = await EnviarSenha(senha, grupo);

            if (resposta.Contains("##")) // Verifica se a resposta contém as hashtags
            {
                Console.WriteLine($"Resposta válida recebida: {resposta}");
                break;
            }
        }

        if (!resposta.Contains("##"))
        {
            Console.WriteLine("Não foi possível obter uma resposta válida após 100 tentativas.");
        }
    }

    static string GerarSenha()
    {
        const string letras = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        const string numeros = "0123456789";
        Random random = new Random();
        StringBuilder senha = new StringBuilder();

        // Adiciona uma letra aleatória
        senha.Append(letras[random.Next(letras.Length)]);
        // Adiciona um número aleatório
        senha.Append(numeros[random.Next(numeros.Length)]);
        // Adiciona mais dois caracteres aleatórios (letra ou número)
        for (int i = 0; i < 2; i++)
        {
            if (random.Next(2) == 0)
                senha.Append(letras[random.Next(letras.Length)]);
            else
                senha.Append(numeros[random.Next(numeros.Length)]);
        }

        return senha.ToString();
    }

    static async Task<string> EnviarSenha(string senha, string grupo)
    {
        using (HttpClient client = new HttpClient())
        {
            var url = "https://fiapnet.azurewebsites.net/fiap";
            var data = new
            {
                Key = senha,
                grupo = grupo
            };
            var json = JsonSerializer.Serialize(data);
            var content = new StringContent(json, Encoding.UTF8, "application/json");

            HttpResponseMessage response = await client.PostAsync(url, content);
            response.EnsureSuccessStatusCode();

            string responseBody = await response.Content.ReadAsStringAsync();
            return responseBody;
        }
    }
}
