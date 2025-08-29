// duplicate-file-finder.ts
import * as fs from "fs";
import * as path from "path";
import * as crypto from "crypto";

function getAllFiles(dir: string, fileList: string[] = []): string[] {
  const files = fs.readdirSync(dir);
  files.forEach(file => {
    const filePath = path.join(dir, file);
    const stat = fs.statSync(filePath);
    if (stat.isDirectory()) {
      getAllFiles(filePath, fileList);
    } else {
      fileList.push(filePath);
    }
  });
  return fileList;
}

function hashFile(filePath: string): string {
  const fileBuffer = fs.readFileSync(filePath);
  const hashSum = crypto.createHash("md5");
  hashSum.update(fileBuffer);
  return hashSum.digest("hex");
}

function findDuplicates(dir: string): void {
  console.log(`🔍 Scanning directory: ${dir}`);
  const files = getAllFiles(dir);

  const hashMap: Record<string, string[]> = {};

  files.forEach(file => {
    const fileHash = hashFile(file);
    if (!hashMap[fileHash]) {
      hashMap[fileHash] = [];
    }
    hashMap[fileHash].push(file);
  });

  let duplicateCount = 0;

  console.log("\n📂 Duplicate Files Found:");
  Object.values(hashMap).forEach(duplicates => {
    if (duplicates.length > 1) {
      duplicateCount++;
      console.log(`\nGroup ${duplicateCount}:`);
      duplicates.forEach(file => console.log(` - ${file}`));
    }
  });

  if (duplicateCount === 0) {
    console.log("\n✅ No duplicate files found!");
  }
}

// Example usage
const targetDir = process.argv[2] || __dirname; // default current folder
findDuplicates(targetDir);
