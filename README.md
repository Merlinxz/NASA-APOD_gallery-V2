# NASA-APOD_gallery
https://merlinxz.github.io/Sklvs/

// Constants for API access and date calculations
const API_KEY = 'BzWsuhSOMGcUpRcxE7OkpVBSVeiPtCIpoFJVq7FO';
const API_URL = 'https://api.nasa.gov/planetary/apod';
const START_DATE = new Date('1995-06-16');
let currentLanguage = 'en'; // Default language is English

document.addEventListener('DOMContentLoaded', () => {
    initApp();
    handleDateFromURL();
    initShareButton();
    initCopyLinkButton();
    document.getElementById('copyDescription').addEventListener('click', copyDescriptionToClipboard);
    document.getElementById('toggleEN').addEventListener('click', () => changeLanguage('en'));
    document.getElementById('toggleTH').addEventListener('click', () => changeLanguage('th'));
});

function initApp() {
    const form = document.getElementById('dateForm');
    const randomImageButton = document.getElementById('randomImage');
    const descriptionModal = new bootstrap.Modal(document.getElementById('descriptionModal'));
    const dateInput = document.getElementById('dateInput');
    initDatePicker(dateInput);

    form.addEventListener('submit', event => handleFormSubmit(event, dateInput));
    randomImageButton.addEventListener('click', () => handleRandomImage(dateInput));
    document.addEventListener('click', event => handleModalDisplay(event, descriptionModal));

    AOS.init({
        duration: 2500,
        easing: 'ease-in-out-quart',
        once: true,
        mirror: false,
    });
}

function initDatePicker(dateInput) {
    const endDate = new Date();
    $(dateInput).datepicker({
        format: 'yyyy-mm-dd',
        startDate: START_DATE,
        endDate: endDate,
        autoclose: true,
        clearBtn: true,
        todayHighlight: true
    }).on('changeDate', function(e) {
        dateInput.value = e.format(0, "yyyy-mm-dd");
        fetchAndDisplayImage(dateInput.value);
        displayDateLabel(dateInput.value);
    });
}

function handleDateFromURL() {
    const queryParams = new URLSearchParams(window.location.search);
    const date = queryParams.get('date');
    if (date) {
        fetchAndDisplayImage(date);
        document.getElementById('dateInput').value = date;
        displayDateLabel(date);
    } else {
        showPictureOfTheDay();
    }
}

function showPictureOfTheDay() {
    const today = new Date().toISOString().split('T')[0];
    fetchAndDisplayImage(today);
}

async function handleFormSubmit(event, dateInput) {
    event.preventDefault();
    const date = dateInput.value || new Date().toISOString().split('T')[0];
    fetchAndDisplayImage(date);
    displayDateLabel(date);
}

async function handleRandomImage(dateInput) {
    const randomDate = getRandomDate();
    fetchAndDisplayImage(randomDate);
    dateInput.value = randomDate;
    displayDateLabel(randomDate);
}

async function fetchAndDisplayImage(date) {
    const nasaContent = document.getElementById('nasaContent');
    nasaContent.innerHTML = '<p>Loading...</p>';
    try {
        const imageData = await fetchImage(date);
        displayContent(imageData);
    } catch (error) {
        nasaContent.innerHTML = `<p class="text-warning">${error.message}</p>`;
    }
}

async function fetchImage(date) {
    const response = await fetch(`${API_URL}?api_key=${API_KEY}&date=${date}`);
    if (!response.ok) {
        throw new Error(`Failed to fetch APOD: ${response.status} ${response.statusText}`);
    }
    return await response.json();
}

function displayContent(data) {
    const nasaContent = document.getElementById('nasaContent');
    const media = data.media_type === 'image' ? `<img src="${data.url}" alt="${data.title}" class="img-fluid">` : `<iframe src="${data.url}" frameborder="0" allowfullscreen class="img-fluid"></iframe>`;
    const contentHtml = `
        <div class="col-12" data-aos="fade-up">
            <h2>${data.title}</h2>
            ${media}
            <button id="showDescription" class="btn btn-outline-light mt-3" data-en="Show Description" data-th="โชว์คำอธิบาย">Show Description</button>
        </div>`;

    nasaContent.innerHTML = contentHtml;
    document.getElementById('modalDescription').innerText = data.explanation;
    AOS.refresh();
}

function handleModalDisplay(event, descriptionModal) {
    if (event.target && event.target.id === 'showDescription') {
        descriptionModal.show();
    }
}

function initShareButton() {
    const shareButton = document.getElementById('shareButton');
    if (navigator.share) {
        shareButton.style.display = 'block';
        shareButton.addEventListener('click', async() => {
            try {
                const title = document.getElementById('descriptionModalLabel').innerText;
                const text = document.getElementById('modalDescription').innerText;
                const imageElement = document.querySelector('#nasaContent img');
                const url = imageElement ? imageElement.src : ''; // Traditional check instead of optional chaining
                await navigator.share({ title, text, url });
                console.log('Content shared successfully!');
            } catch (error) {
                console.error('Error sharing:', error);
                alert('Failed to share content.');
            }
        });
    } else {
        shareButton.style.display = 'none'; // Hide button if Web Share is not supported
    }
}

function initCopyLinkButton() {
    const copyLinkButton = document.getElementById('copyLink');
    copyLinkButton.addEventListener('click', () => {
        const dateInput = document.getElementById('dateInput').value;
        const urlWithDate = `${window.location.origin}${window.location.pathname}?date=${dateInput}`;
        navigator.clipboard.writeText(urlWithDate).then(() => {
            console.log('Link copied to clipboard');
            alert('Link copied to clipboard!');
        }).catch(err => {
            console.error('Failed to copy link:', err);
            alert('Failed to copy link.');
        });
    });
}

function copyDescriptionToClipboard() {
    const descriptionText = document.getElementById('modalDescription').innerText;
    navigator.clipboard.writeText(descriptionText).then(() => {
        console.log('Description copied to clipboard');
        alert('Description copied to clipboard!');
    }).catch(err => {
        console.error('Failed to copy text: ', err);
        alert('Failed to copy description.');
    });
}

function getRandomDate() {
    const end = new Date();
    const randomTime = START_DATE.getTime() + Math.random() * (end.getTime() - START_DATE.getTime());
    return new Date(randomTime).toISOString().split('T')[0];
}

function displayDateLabel(date) {
    const dateLabel = document.getElementById('dateLabel');
    if (!dateLabel) {
        createLabelElement(date);
    } else {
        dateLabel.innerText = `Selected Date: ${date}`;
    }
}

function createLabelElement(date) {
    const label = document.createElement('label');
    label.id = 'dateLabel';
    label.innerText = `Selected Date: ${date}`;
    label.style.color = 'var(--accent-color)';
    label.style.marginTop = '10px';
    document.querySelector('.container').appendChild(label);
}

function changeLanguage(lang) {
    const elements = document.querySelectorAll('[data-en], [data-th]');
    elements.forEach(element => {
        if (lang === 'en') {
            element.innerText = element.getAttribute('data-en');
        } else if (lang === 'th') {
            element.innerText = element.getAttribute('data-th');
        }
    });
}
