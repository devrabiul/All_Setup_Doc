$('#generateAndDownloadSitemap').on('click', function (){
    let getRoute = $(this).data('route');
    $.ajaxSetup({
        headers: {
            "X-CSRF-TOKEN": $('meta[name="_token"]').attr("content"),
        },
    });
    $.ajax({
        type: "get",
        url: getRoute,
        beforeSend: function () {
            $('#generateAndDownloadSitemapSpinner').show();
            $(this).attr('disabled', true);
        },
        success: function (response) {
            downloadFileUsingFileUrl(response.filePath)
            if (response.status === 1) {
                location.reload();
            }
        },
        error: function () {
            $('#generateAndDownloadSitemapSpinner').hide();
            $(this).attr('disabled', false);
        },
        complete: function () {
            $('#generateAndDownloadSitemapSpinner').hide();
            $(this).attr('disabled', false);
            location.reload();
        },
    });
})

function downloadFileUsingFileUrl(url) {
    fetch(url)
        .then(response => response.blob())
        .then(blob => {
            const filename = url.substring(url.lastIndexOf('/') + 1);
            const blobUrl = window.URL.createObjectURL(new Blob([blob]));
            const link = document.createElement('a');
            link.href = blobUrl;
            link.setAttribute('download', filename);
            document.body.appendChild(link);
            link.click();
            link.parentNode.removeChild(link);
        })
        .catch(error => console.error('Error downloading file:', error));
}
