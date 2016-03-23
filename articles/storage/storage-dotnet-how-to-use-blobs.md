<properties
	pageTitle="Introdução ao armazenamento de Blobs do Azure usando o .NET | Microsoft Azure"
	description="Armazene dados de arquivos na nuvem com o Armazenamento de Blobs do Azure (armazenamento de objeto). Obtenha uma introdução a operações de armazenamento de Blobs simples, incluindo a criação de um contêiner e o carregamento, o download, a listagem e a exclusão do conteúdo de blobs."
	services="storage"
	documentationCenter=".net"
	authors="tamram"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.date="02/25/2016"
	ms.author="tamram"/>


# Introdução ao armazenamento de Blobs do Azure usando o .NET

[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## Visão geral

O armazenamento de Blobs do Azure é um serviço que armazena dados de arquivos na nuvem. O armazenamento de blobs pode ser qualquer tipo de texto ou dados binários, como um documento, um arquivo de mídia ou um instalador do aplicativo. O Armazenamento de Blobs também é chamado de armazenamento de objeto.

### Sobre este tutorial

Este tutorial mostra como gravar código .NET para alguns cenários comuns usando o armazenamento de Blobs do Azure. Os cenários abrangidos incluem carregamento, listagem, download e exclusão de blobs.

**Tempo estimado para conclusão:** 45 minutos

**Pré-requisitos:**

- [Microsoft Visual Studio](https://www.visualstudio.com/pt-BR/visual-studio-homepage-vs.aspx)
- [Biblioteca do Cliente de Armazenamento do Azure para .NET](https://www.nuget.org/packages/WindowsAzure.Storage/)
- [Gerenciador de configuração do Azure para .NET](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
- Uma [conta de armazenamento do Azure](storage-create-storage-account.md#create-a-storage-account).


[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [armazenamento-blob-conceitos-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-configure-connection-string-include](../../includes/storage-configure-connection-string-include.md)]

## Acessar programaticamente o armazenamento de blobs

[AZURE.INCLUDE [storage-dotnet-obtain-assembly](../../includes/storage-dotnet-obtain-assembly.md)]

### Declarações de namespace

Adicione as seguintes declarações de namespace à parte superior de qualquer arquivo C# no qual você deseja acessar o armazenamento do Azure programaticamente:

    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Blob;

Certifique-se de fazer referência ao assembly `Microsoft.WindowsAzure.Storage.dll`.

[AZURE.INCLUDE [storage-dotnet-retrieve-conn-string](../../includes/storage-dotnet-retrieve-conn-string.md)]

Um tipo **CloudBlobClient** permite que você recupere objetos que representam contêineres e blobs armazenados no Serviço de Armazenamento de Blobs. O código a seguir cria um objeto **CloudBlobClient** usando o objeto da conta de armazenamento que recuperamos acima:

    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

## Criar um contêiner

[AZURE.INCLUDE [armazenamento-contêiner-nomeando-regras-include](../../includes/storage-container-naming-rules-include.md)]

Este exemplo mostra como criar um contêiner se ainda não existir:

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve a reference to a container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Create the container if it doesn't already exist.
    container.CreateIfNotExists();

Por padrão, o novo contêiner é privado e você deve especificar sua chave de acesso do armazenamento para baixar blobs desse contêiner. Para disponibilizar arquivos dentro do contêiner a todas as pessoas, você pode definir o contêiner como público usando o seguinte código:

    container.SetPermissions(
        new BlobContainerPermissions { PublicAccess =
 	    BlobContainerPublicAccessType.Blob });

Qualquer pessoa na Internet pode ver blobs em um contêiner público, mas você só pode modificar ou excluí-los se tiver a chave de acesso apropriada.

## Carregar um blob em um contêiner

O Armazenamento de Blob do Azure oferece suporte a blobs de blocos e a blobs de páginas. Na maioria dos casos, o blob de blocos é o tipo recomendado a ser usado.

Para carregar um arquivo em um blob de blocos, obtenha uma referência de contêiner e use-a para obter uma referência de blob de blocos. Quando tiver uma referência de blob, poderá transferir qualquer fluxo de dados para ele, chamando o método **UploadFromStream**. Essa operação criará o blob, caso ele não exista, ou o substituirá, caso ele já exista.

O exemplo a seguir mostra como carregar um blob em um contêiner e pressupõe que o contêiner já tenha sido criado.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Retrieve reference to a blob named "myblob".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

    // Create or overwrite the "myblob" blob with contents from a local file.
    using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
    {
        blockBlob.UploadFromStream(fileStream);
    }

## Listar os blobs em um contêiner

Para listar blobs em um contêiner, primeiro obtenha uma referência ao contêiner. Você pode usar o método **ListBlobs** do contêiner para recuperar os blobs e/ou diretórios dentro dele. Para acessar o avançado conjunto de propriedades e métodos para um **IListBlobItem** retornado, você deve convertê-lo em um objeto **CloudBlockBlob**, **CloudPageBlob** ou **CloudBlobDirectory**. Se o tipo for desconhecido, você poderá usar uma verificação de tipo para determinar no qual convertê-lo. O código a seguir demonstra como recuperar e apresentar a saída do URI de cada item no `photos`contêiner:

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
	CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	// Retrieve reference to a previously created container.
	CloudBlobContainer container = blobClient.GetContainerReference("photos");

	// Loop over items within the container and output the length and URI.
	foreach (IListBlobItem item in container.ListBlobs(null, false))
	{
		if (item.GetType() == typeof(CloudBlockBlob))
		{
			CloudBlockBlob blob = (CloudBlockBlob)item;

			Console.WriteLine("Block blob of length {0}: {1}", blob.Properties.Length, blob.Uri);

		}
		else if (item.GetType() == typeof(CloudPageBlob))
		{
			CloudPageBlob pageBlob = (CloudPageBlob)item;

			Console.WriteLine("Page blob of length {0}: {1}", pageBlob.Properties.Length, pageBlob.Uri);

		}
		else if (item.GetType() == typeof(CloudBlobDirectory))
		{
			CloudBlobDirectory directory = (CloudBlobDirectory)item;

			Console.WriteLine("Directory: {0}", directory.Uri);
		}
	}

Como mostrado acima, você pode nomear blobs com informações de caminho em seus nomes. Isso cria uma estrutura de diretório virtual que você pode organizar e percorrer como faria com um sistema de arquivos tradicional. Observe que a estrutura do diretório é virtual apenas - os somente os recursos disponíveis no armazenamento de Blob são contêineres e blobs. No entanto, a biblioteca de cliente de armazenamento oferece um objeto **CloudBlobDirectory** para fazer referência a um diretório virtual e simplificar o processo e trabalhar com blobs que são organizados dessa maneira.

Por exemplo, considere o seguinte conjunto de blobs de blocos em um contêiner chamado`photos`:

	photo1.jpg
	2010/architecture/description.txt
	2010/architecture/photo3.jpg
	2010/architecture/photo4.jpg
	2011/architecture/photo5.jpg
	2011/architecture/photo6.jpg
	2011/architecture/description.txt
	2011/photo7.jpg

Quando você chama **ListBlobs** no contêiner 'photos' (como no exemplo acima), uma listagem hierárquica é retornada. Ela contém os objetos **CloudBlobDirectory** e **CloudBlockBlob**, que representam os diretórios e blobs no contêiner, respectivamente. A saída resultante é semelhante a:

	Directory: https://<accountname>.blob.core.windows.net/photos/2010/
	Directory: https://<accountname>.blob.core.windows.net/photos/2011/
	Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg


Opcionalmente, você pode definir o parâmetro **UseFlatBlobListing** do método **ListBlobs** como **true**. Nesse caso, cada blob no contêiner é retornado como um objeto **CloudBlockBlob**. A chamada para **ListBlobs** para retornar uma lista simples tem esta aparência:

    // Loop over items within the container and output the length and URI.
	foreach (IListBlobItem item in container.ListBlobs(null, true))
	{
	   ...
	}

e os resultados têm esta aparência:

	Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2010/architecture/description.txt
	Block blob of length 314618: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo3.jpg
	Block blob of length 522713: https://<accountname>.blob.core.windows.net/photos/2010/architecture/photo4.jpg
	Block blob of length 4: https://<accountname>.blob.core.windows.net/photos/2011/architecture/description.txt
	Block blob of length 419048: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo5.jpg
	Block blob of length 506388: https://<accountname>.blob.core.windows.net/photos/2011/architecture/photo6.jpg
	Block blob of length 399751: https://<accountname>.blob.core.windows.net/photos/2011/photo7.jpg
	Block blob of length 505623: https://<accountname>.blob.core.windows.net/photos/photo1.jpg


## Baixar blobs

Para baixar blobs, primeiro recupere uma referência a um blob e, em seguida, chame o método **DownloadToStream**. O exemplo a seguir usa o método **DownloadToStream** para transferir o conteúdo do blob para um objeto de fluxo que você pode persistir em um arquivo local.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Retrieve reference to a blob named "photo1.jpg".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

    // Save blob contents to a file.
    using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
    {
        blockBlob.DownloadToStream(fileStream);
    }

Você também pode usar o método **DownloadToStream** para baixar o conteúdo de um blob como uma cadeia de texto.

	// Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

	// Retrieve reference to a blob named "myblob.txt"
	CloudBlockBlob blockBlob2 = container.GetBlockBlobReference("myblob.txt");

	string text;
	using (var memoryStream = new MemoryStream())
	{
		blockBlob2.DownloadToStream(memoryStream);
		text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
	}

## Excluir blobs

Para excluir um blob, primeiro obtenha uma referência de blob e, em seguida, chame o método **Delete**.

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Retrieve reference to a blob named "myblob.txt".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

    // Delete the blob.
    blockBlob.Delete();


## Listar blobs em páginas de maneira assíncrona

Se você está listando uma grande quantidade de blobs ou se deseja controlar o número de resultados retornados em uma operação de listagem, pode listar os blobs em páginas de resultados. Este exemplo mostra como retornar resultados em páginas de forma assíncrona, para que a execução não fique bloqueada enquanto espera para retornar um grande conjunto de resultados.

Este exemplo mostra uma listagem de blob simples, mas você também pode realizar uma listagem hierárquica, configurando o `useFlatBlobListing` parâmetro do método **ListBlobsSegmentedAsync** como `false`.

Como o método de amostra chama um método assíncrono, ele deve ser precedido pela `async` palavra-chave e deve retornar um objeto **Task**. A palavra-chave await especificada para o método **ListBlobsSegmentedAsync** suspende a execução do método de amostra até que a tarefa de listagem seja concluída.

    async public static Task ListBlobsSegmentedInFlatListing(CloudBlobContainer container)
    {
        //List blobs to the console window, with paging.
        Console.WriteLine("List blobs in pages:");

        int i = 0;
        BlobContinuationToken continuationToken = null;
        BlobResultSegment resultSegment = null;

        //Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
        //When the continuation token is null, the last page has been returned and execution can exit the loop.
        do
        {
            //This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
            //or by calling a different overload.
            resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);
            if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
            foreach (var blobItem in resultSegment.Results)
            {
                Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
            }
            Console.WriteLine();

            //Get the continuation token.
            continuationToken = resultSegment.ContinuationToken;
        }
        while (continuationToken != null);
    }

## Gravar um blob de anexo

Um blob de anexo é um novo tipo de blob, introduzido com a versão 5.x da biblioteca de cliente de armazenamento do Azure para .NET. Um blob de anexo é otimizado para operações de anexo, como registro em log. Como um blob de blocos, um blob de anexo é composto de blocos; no entanto, quando você adiciona um novo bloco a um blob de anexo, ele sempre é acrescentado ao fim do blob. Não é possível atualizar ou excluir um bloco existente em um blob de anexo. As IDs de bloco para um blob de anexo não ficam expostas como para um blob de blocos.

Cada bloco em um blob de anexo pode ter um tamanho diferente, até no máximo 4 MB, e um blob de anexo pode incluir no máximo 50.000 blocos. O tamanho máximo de um blob de anexo, portanto, é de pouco mais de 195 GB (4 MB x 50.000 blocos).

O exemplo a seguir cria um novo blob de anexo e acrescenta alguns dados a ele, simulando uma operação simples de registro em log.

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    //Create service client for credentialed access to the Blob service.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    //Get a reference to a container.
    CloudBlobContainer container = blobClient.GetContainerReference("my-append-blobs");

    //Create the container if it does not already exist.
    container.CreateIfNotExists();

    //Get a reference to an append blob.
    CloudAppendBlob appendBlob = container.GetAppendBlobReference("append-blob.log");

    //Create the append blob. Note that if the blob already exists, the CreateOrReplace() method will overwrite it.
    //You can check whether the blob exists to avoid overwriting it by using CloudAppendBlob.Exists().
    appendBlob.CreateOrReplace();

    int numBlocks = 10;

    //Generate an array of random bytes.
    Random rnd = new Random();
    byte[] bytes = new byte[numBlocks];
    rnd.NextBytes(bytes);

    //Simulate a logging operation by writing text data and byte data to the end of the append blob.
    for (int i = 0; i < numBlocks; i++)
    {
        appendBlob.AppendText(String.Format("Timestamp: {0:u} \tLog Entry: {1}{2}",
            DateTime.UtcNow, bytes[i], Environment.NewLine));
    }

    //Read the append blob to the console window.
    Console.WriteLine(appendBlob.DownloadText());

Para saber mais sobre as diferenças entre os três tipos de blobs, confira [Noções gerais sobre Blobs de Blocos, Blobs de Páginas e Blobs de Anexo](https://msdn.microsoft.com/library/azure/ee691964.aspx).

## Próximas etapas

Agora que você aprendeu os conceitos básicos do armazenamento de Blob, siga estes links para saber mais.

### Documentação de referência do armazenamento de Blob

- [Referência à Biblioteca de Cliente de Armazenamento para .NET](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
- [Referência da API REST](http://msdn.microsoft.com/library/azure/dd179355)

### Guias de recursos adicionais

- [Introdução ao Armazenamento de tabelas para .NET](storage-dotnet-how-to-use-tables.md)
- [Introdução ao Armazenamento de filas para .NET](storage-dotnet-how-to-use-queues.md)
- [Introdução ao Armazenamento de arquivos para .NET](storage-dotnet-how-to-use-files.md)
- [Transferir dados com o utilitário de linha de comando AzCopy](storage-use-azcopy)
- [Como usar o armazenamento de blob do Azure com o SDK de Trabalhos Web](../app-service-web/websites-dotnet-webjobs-sdk-storage-blobs-how-to.md)
- [Usando o Banco de Dados SQL do .NET (C#)](../sql-database/sql-database-dotnet-how-to-use.md)

  [Blob5]: ./media/storage-dotnet-how-to-use-blobs/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-blobs/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-blobs/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-blobs/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-blobs/blob9.png

  [Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
  [Configuring Connection Strings]: http://msdn.microsoft.com/library/azure/ee758697.aspx
  [.NET client library reference]: http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
  [REST API reference]: http://msdn.microsoft.com/library/azure/dd179355

<!---HONumber=AcomDC_0309_2016-->
