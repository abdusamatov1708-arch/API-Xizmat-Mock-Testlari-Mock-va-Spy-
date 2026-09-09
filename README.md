# API-Xizmat-Mock-Testlari-Mock-va-Spy-
/**
 * API Xizmat Moduli
 * Tashqi so'rovlar va bildirishnomalar bilan ishlovchi funksiyalar
 */

async function fetchUser(id) {
    const response = await fetch(`https://api.example.com/users/${id}`);
    if (!response.ok) {
        throw new Error("Foydalanuvchini olishda xatolik");
    }
    return response.json();
}

async function saveUser(user) {
    const response = await fetch('https://api.example.com/users', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(user)
    });
    if (!response.ok) {
        throw new Error("Foydalanuvchini saqlashda xatolik");
    }
    return response.json();
}

function notifyAdmin(xabar) {
    console.warn(`Ogohlantirish: ${xabar}`);
}

module.exports = {
    fetchUser,
    saveUser,
    notifyAdmin
};
Фрагмент кода
const { fetchUser, saveUser, notifyAdmin } = require('./apiService');

// Global fetch ni moklash
global.fetch = jest.fn();

describe("API Xizmat Testlari (Mock va Spy)", () => {

    afterEach(() => {
        jest.clearAllMocks();
    });

    describe("fetchUser funksiyasi", () => {
        test("muvaffaqiyatli foydalanuvchi ma'lumotini olish (mockResolvedValue)", async () => {
            const mockUser = { id: 1, name: "Ali" };

            global.fetch.mockResolvedValueOnce({
                ok: true,
                json: async () => mockUser
            });

            const user = await fetchUser(1);
            expect(global.fetch).toHaveBeenCalledTimes(1);
            expect(global.fetch).toHaveBeenCalledWith("https://api.example.com/users/1");
            expect(user).toEqual(mockUser);
        });

        test("tarmoq yoki server xatosi holati (mockRejectedValue)", async () => {
            global.fetch.mockRejectedValueOnce(new Error("Tarmoqda xatolik"));

            await expect(fetchUser(2)).rejects.toThrow("Tarmoqda xatolik");
            expect(global.fetch).toHaveBeenCalledTimes(1);
        });

        test("javob statusi ok bo'lmaganda xato tashlash", async () => {
            global.fetch.mockResolvedValueOnce({
                ok: false
            });

            await expect(fetchUser(99)).rejects.toThrow("Foydalanuvchini olishda xatolik");
        });
    });

    describe("saveUser funksiyasi", () => {
        test("foydalanuvchini muvaffaqiyatli saqlash (POST so'rovi)", async () => {
            const newUser = { name: "Vali", age: 25 };
            const savedResponse = { id: 2, ...newUser };

            global.fetch.mockResolvedValueOnce({
                ok: true,
                json: async () => savedResponse
            });

            const result = await saveUser(newUser);
            expect(global.fetch).toHaveBeenCalledTimes(1);
            expect(global.fetch).toHaveBeenCalledWith(
                "https://api.example.com/users",
                expect.objectContaining({
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(newUser)
                })
            );
            expect(result).toEqual(savedResponse);
        });
    });

    describe("notifyAdmin funksiyasi (jest.spyOn)", () => {
        test("console.warn chaqirilganini va to'g'ri xabar berganini kuzatish", () => {
            const spy = jest.spyOn(console, "warn").mockImplementation(() => {});

            notifyAdmin("Tizimda xatolik yuz berdi");

            expect(spy).toHaveBeenCalled();
            expect(spy).toHaveBeenCalledWith("Ogohlantirish: Tizimda xatolik yuz berdi");

            spy.mockRestore();
        });
    });

});
